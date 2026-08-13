# Sonho dá Sorte

Repositório do site estático da marca. São **duas coisas** no mesmo domínio:

- **`/`** → o **Totem** (tela de sorteio animada, retrato 1080×1920).
- **`/links`** → a página **link-in-bio** da marca.

Nada de build: é HTML estático, sobe em qualquer host (Netlify, Vercel, GitHub Pages, S3…) e já funciona.

---

## Estrutura

```
.
├── index.html            ← TOTEM (resolve em /) — autocontido, roda offline
├── vendor/               ← libs locais do totem (sem depender de CDN)
│   ├── vue.global.prod.js
│   └── lottie.min.js
├── links/
│   └── index.html        ← página link-in-bio (resolve em /links)
├── docs/
│   └── PLANO-DE-EVOLUCAO.md   ← spec do app real de /links (Vue+Quasar+Supabase)
├── src-vue/
│   └── TotemSonhoSorte.vue    ← componente do totem p/ integrar no projeto Quasar
├── assets/               ← imagens + Lotties usados pelo componente .vue
│   ├── halo.webp  bg_cosmic.jpg  trophy.png  logo.png  qr.svg  clover.png
│   ├── coins/coin1..8.webp
│   └── lottie/neutral.json  blink/coins/cool/fun/happy.json
├── .gitignore
└── README.md
```

---

## Como publicar

**Netlify / Vercel:** conecte o repositório, **sem build command**, publish directory = raiz. `/` mostra o totem e `/links` a página.

**GitHub Pages:** Settings → Pages → branch `main`, pasta `/ (root)`. Totem em `.../` e página em `.../links/`. Com domínio próprio: `https://SEU-DOMINIO/` e `https://SEU-DOMINIO/links`.

> O totem **não depende de internet/CDN**: o Vue e o lottie-web ficam em `/vendor` e todos os assets estão embutidos no `index.html`. Ideal para totem em rede fechada.

---

## O Totem (`index.html`)

Tela de sorteio que **cicla sozinha**: revela o número (bolas girando e travando), calcula os prêmios (Sena→Dupla como sufixos do número), conta os ganhadores, roda os contadores e reinicia. O mascote **Lucky** é uma Lottie (pose neutra em loop + reações aleatórias).

### Ajustes rápidos — bloco `CONFIG` no topo do `<script>`

| Chave | O que faz | Padrão |
|---|---|---|
| `drawIntervalSec` | tempo entre um sorteio e o próximo | `20` (demo; produção ~`600`) |
| `spinMs` / `gapMs` | duração do giro de cada bola / intervalo entre elas | `700` / `180` |
| `digits` | quantidade de dígitos sorteados | `6` |
| `drawTime` | horário do sorteio exibido na pílula (estático) | `'10:00'` |
| `luckySpeed` | velocidade da pose **idle** do Lucky | `0.5` |
| `reactionSpeed` | velocidade das **reações** do Lucky | `0.75` |
| `reactionEverySec` | dispara uma reação a cada X segundos | `5` |
| `nextDrawSec` / `nextDrawLabel` | contador longo do card e seu rótulo | `24:48` / `'Hoje às 11h'` |

**Trocar o mock por API real:** hoje o número, ganhadores e nº do sorteio são gerados no cliente (funções `drawNumber` / `animateWinners`). Basta substituir essas funções por uma leitura da sua API/websocket, mantendo os mesmos nomes reativos (`balls`, `tiers`, `winnersShown`, `sorteioNum`, etc.) — o template não muda.

### Lucky (Lottie)
As poses estão em `assets/lottie/`. O mecanismo (no `onMounted`) toca a `neutral` em loop e, a cada `reactionEverySec`, sorteia uma de `blink/coins/cool/fun/happy`, toca 1×, e volta pro idle. Para adicionar/remover poses, edite o array `LUCKY.reactions`.

---

## Integração no Quasar (`src-vue/`)

`TotemSonhoSorte.vue` é o mesmo totem como Single File Component. No seu projeto:

1. `npm i lottie-web`
2. Copie a pasta `assets/` para junto do componente (ele referencia `./assets/...` via `new URL(..., import.meta.url)`; o Vite/Quasar resolve JSON e binários direto).
3. Importe e use `<TotemSonhoSorte />` numa rota/página em tela cheia.

> O `index.html` da raiz é a **build estática** já pronta pra deploy; o `.vue` é a **fonte** pra você evoluir no Quasar. Os dois têm o mesmo comportamento.

---

## A página `/links`

Versão **estática (preview)**: HTML autocontido (logo em base64), sem back-end. Serve pra publicar e validar visual/fluxo. A versão completa (com o interpretador de sonhos e painel de edição de botões) é construída a partir de `docs/PLANO-DE-EVOLUCAO.md` (Vue+Quasar+Supabase).

### Pendências antes de ir ao ar
1. **Botão "Participe do próximo sorteio"** está com link placeholder `#` — editar em `links/index.html`.
2. **Copy regulatória:** revisar textos para não prometer/insinuar ganho (sorteio, SUSEP, +18).
