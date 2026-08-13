# Sonho dá Sorte — Plano de Evolução (spec executável por blocos)

> **Para o Claude Code.** As decisões já foram tomadas na fase de estratégia; aqui só se executa. Este é o `ROADMAP.md` do kit de construção. O visual é fiel ao Figma entregue pelo dono (não reinventar layout).

## O que é o produto
Página única de link-in-bio da marca **Sonho dá Sorte** (não é produto multiusuário — é UMA página, de UMA marca). Funil: sorteio → interpretação de sonho → app.

- **Alma / "e agora?":** cada bloco leva a um próximo passo óbvio (participar, interpretar, baixar).
- **Diferencial (peça central):** o **Lucky**, interpretador de sonhos. Não é "mais um link" — é o coração do produto.
- **Restrição de negócio/regulatória:** produto de sorteio, **+18**, exibir "jogue com responsabilidade". Copy **não pode prometer/insinuar ganho** ("você vai ganhar", "mudança de vida garantida"). Subtítulo do CTA principal já foi ajustado para linguagem sem promessa.

## Stack (fechada)
- **Front:** Vue 3 + **Quasar**. Duas superfícies: página pública + painel de edição do dono.
- **Back:** **Supabase** (Postgres gerenciado + Auth + API automática, RLS ligada por padrão).
- **Deploy:** automático no git push. Repo privado.
- **Segredos:** só no servidor. Nada de service key no cliente.

## Modelo de dados (mínimo)
Tabela `links`:
- `id` (uuid)
- `titulo` (text)
- `subtitulo` (text, opcional)
- `url` (text)
- `ordem` (int) — ordena a lista na página
- `ativo` (bool) — liga/desliga o botão sem apagar
- `destaque` (bool) — o botão verde principal ("Participe do próximo sorteio")
- `created_at`, `updated_at`

Se o Lucky guardar o texto do sonho, isso é **dado pessoal (LGPD)** → tabela separada, com minimização, retenção definida e base legal registrada. Ver Bloco 8.

## Convenções (valem sem perguntar)
- Migrações **sequenciais, uma por assunto, nunca destrutivas**. Campo em desuso vira órfão (para de ser lido), não é removido enquanto houver dado.
- Autorização a nível de linha em toda tabela nova.
- **Exceção pública:** a leitura dos links na página aberta é a única superfície sem login — política própria, **só SELECT de `ativo = true`**, nunca INSERT/UPDATE/DELETE, nunca enumeração de outras tabelas.
- Validação consistente de entrada; erro genérico ao usuário, log detalhado interno.

## Blocos

**Bloco 0 — Preflight.** Projeto Supabase criado, env vars verdes, Quasar rodando local, repo + deploy automático conectados.

**Bloco 1 — Esqueleto + 1º deploy.** App Quasar no ar (mesmo que "hello world"), deploy no git push funcionando.

**Bloco 2 — Banco + política pública.** *(revisão de segurança)* Criar tabela `links` com RLS. Política pública: leitura de `ativo = true`, nada mais. Seed com os 3 botões atuais (Participe / Descubra o que seu sonho significa / Baixe o app).

**Bloco 3 — Auth do dono.** *(revisão de segurança)* Login só do dono da marca (não é cadastro público). Painel protegido por auth; RLS garante que só o dono escreve em `links`.

**Bloco 4 — Layout base + design tokens (do Figma).**
Extraídos do Figma entregue — usar como tokens, não hardcode espalhado:
- **Fundo:** base `#1A0938`; app `radial-gradient(120% 80% at 50% -10%, #4A1A8F 0%, #2D0F5E 45%, #1A0938 100%)`.
- **CTA principal (verde/destaque):** `linear-gradient(100deg, #C7EC4E, #ACE019)`, texto `#3D0F6B`, sombra `0 10px 30px rgba(255,184,0,0.35)`, radius 16px.
- **Cards secundários:** fundo `rgba(255,255,255,0.06)`, borda `rgba(162,89,255,0.4)`, radius 16px.
- **Pílula "Sorteios das 08h às 23h":** fundo `rgba(248,217,42,0.14)`, borda `rgba(248,217,42,0.5)`, texto `#F8D92A`.
- **Acento:** `#F8D92A` / `#FFCF3F` (sparkles). **Texto:** primário `#F3ECFF`, secundário `#C6B3EC`.
- **Fonte:** Figtree. Larguras mobile-first (~393px), max-width ~480px centralizado.

**Bloco 5 — Página pública (a home que responde "e agora?").** Renderiza logo + slogan "Custa pouco sonhar" + pílula de horário + lista de `links` (ordenada, só ativos, `destaque` = botão verde) + badges de app + ícones sociais + footer "+18 / jogue com responsabilidade". Fiel ao Figma.

**Bloco 6 — Painel de edição dos links.** CRUD simples da tabela `links`: criar, editar título/subtítulo/url, reordenar, ativar/desativar, marcar destaque. É o que dá autonomia pro dono adicionar botões sem dev.

**Bloco 7 — (n/a nesta versão).** Sem intake de dados de terceiros além do painel. Manter reservado.

**Bloco 8 — O diferencial: Lucky (interpretador de sonhos).** *(revisão de segurança + LGPD)*
- IA via API, chave **só no servidor** (proxy), nunca no cliente.
- Instrução de papel: interpreta sonho de forma acolhedora; **proibido prometer resultado de sorteio**.
- Se guardar o texto do sonho: tabela própria, minimização, retenção definida, base legal (consentimento) registrada. Se não precisar guardar, **não guardar** (mais seguro).
- Fluxo "e agora?": pessoa conta o sonho → recebe leitura → é convidada ao sorteio.

**Bloco 9 — Superfícies públicas/sensíveis.** *(revisão de segurança)* Rate-limit na chamada do Lucky (é a porta de entrada mais cara/abusável). Gating +18 antes de fluxos de sorteio. Revisar que a página pública não vaza nada além dos links ativos.

**Bloco 10–12 — Polimento.** Estados vazio/carregando/erro (inclusive Lucky "pensando" e falha de rede), performance (imagens, fontes), acessibilidade dos botões, e revisão final de copy sob a lente regulatória.

## Pendências para o dono confirmar (não travam o começo)
1. O Lucky **guarda** o texto do sonho ou é efêmero? (decide se entra tabela + LGPD)
2. Ícones sociais no Figma parecem placeholder (labels trocados) — confirmar as redes reais e URLs.
3. Subtítulo do CTA principal: aprovar a versão sem promessa ("Concorra a prêmios todo dia." / "Seu palpite pode ser premiado.").
