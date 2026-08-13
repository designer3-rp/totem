<template>

    <div class="viewport">
      <div class="scaler" :style="{ transform: 'translate(-50%,-50%) scale(' + scale + ')' }">
        <div class="stage">

          <!-- fundo -->
          <div class="bg-base"></div>
          <img class="bg-cosmic" :src="assets.bg" alt="">
          <div class="bg-numbers">
            <span v-for="(n,i) in bgNums" :key="'n'+i"
                  :style="{ left:n.x+'%', top:n.y+'%', fontSize:n.s+'px', transform:'rotate('+n.r+'deg)' }">{{ n.t }}</span>
          </div>
          <img class="halo" :src="assets.halo" alt="">
          <img class="coin c1" :src="assets.coin1" alt="">
          <img class="coin c2" :src="assets.coin2" alt="">
          <img class="coin c3" :src="assets.coin3" alt="">
          <img class="coin c4" :src="assets.coin4" alt="">
          <img class="coin c5" :src="assets.coin5" alt="">
          <img class="coin c6" :src="assets.coin6" alt="">
          <img class="coin c7" :src="assets.coin7" alt="">
          <img class="coin c8" :src="assets.coin8" alt="">
          <div class="lucky" ref="luckyEl"></div>

          <!-- header -->
          <header class="hdr">
            <img class="logo" :src="assets.logo" alt="Sonho dá Sorte">
            <div class="hdr-right">
              <div class="date">{{ dateLabel }}</div>
              <div class="clock">{{ clock }}</div>
            </div>
          </header>

          <!-- pílula do sorteio -->
          <div class="pill">
            <div class="pill-left"><img class="clover" :src="assets.clover" alt=""> Sorteio {{ sorteioNum }}</div>
            <div class="pill-right">
              <svg viewBox="0 0 24 24" fill="none"><circle cx="12" cy="12" r="9" stroke-width="2"/><path d="M12 7v5l3 2" stroke-width="2" stroke-linecap="round"/></svg>
              {{ drawTime }}
            </div>
          </div>

          <!-- título -->
          <div class="title"><span class="flourish left"></span><span class="title-text">NÚMERO SORTEADO</span><span class="flourish right"></span></div>

          <!-- bolas -->
          <div class="balls">
            <div v-for="(b,i) in balls" :key="'b'+i" class="ball"
                 :class="{ spin:b.spinning, land:b.landed, glow:b.glow }">
              <span class="digit">{{ b.digit }}</span>
            </div>
          </div>

          <!-- prêmios -->
          <div class="tiers">
            <div v-for="(t,i) in tiers" :key="'t'+i" class="tier" :class="{ show:t.show }" :style="{ '--d': (i*0.09)+'s' }">
              <div class="tier-icon">{{ t.icon }}</div>
              <div class="tier-name">{{ t.name }}</div>
              <div class="tier-value">{{ t.value }}</div>
            </div>
          </div>

          <!-- ganhadores -->
          <div class="winners">
            <img class="trophy" :src="assets.trophy" alt="">
            <div class="winners-txt"><b :class="{ dim: !complete }">{{ winnersShown }}</b> ganhadores neste sorteio</div>
          </div>

          <!-- próximo sorteio -->
          <div class="card">
            <div class="card-kicker">PRÓXIMO SORTEIO</div>
            <div class="card-when">{{ nextLabel }}</div>
            <div class="cd">
              <div class="cd-grp"><span class="cd-num">{{ nextH }}</span><span class="cd-lab">hr</span></div>
              <span class="cd-sep">:</span>
              <div class="cd-grp"><span class="cd-num">{{ nextM }}</span><span class="cd-lab">min</span></div>
              <span class="cd-sep">:</span>
              <div class="cd-grp"><span class="cd-num">{{ nextS }}</span><span class="cd-lab">seg</span></div>
            </div>
            <div class="bar"><div class="bar-fill" :style="{ width:(progress*100)+'%' }"></div></div>
          </div>

          <!-- QR -->
          <img class="qr" :src="assets.qr" alt="QR code">

        </div>
      </div>
    </div>

</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue';
import lottie from 'lottie-web';
import luckyData from './assets/lottie/neutral.json';
import r_coins from './assets/lottie/coins.json';
import r_cool from './assets/lottie/cool.json';
import r_fun from './assets/lottie/fun.json';
import r_happy from './assets/lottie/happy.json';
const luckyReactions = [r_coins, r_cool, r_fun, r_happy];

const assets = {
  bg: new URL('./assets/bg_cosmic.jpg', import.meta.url).href,
  halo: new URL('./assets/halo.webp', import.meta.url).href,
  coin1: new URL('./assets/coins/coin1.webp', import.meta.url).href,
  coin2: new URL('./assets/coins/coin2.webp', import.meta.url).href,
  coin3: new URL('./assets/coins/coin3.webp', import.meta.url).href,
  coin4: new URL('./assets/coins/coin4.webp', import.meta.url).href,
  coin5: new URL('./assets/coins/coin5.webp', import.meta.url).href,
  coin6: new URL('./assets/coins/coin6.webp', import.meta.url).href,
  coin7: new URL('./assets/coins/coin7.webp', import.meta.url).href,
  coin8: new URL('./assets/coins/coin8.webp', import.meta.url).href,
  trophy: new URL('./assets/trophy.png', import.meta.url).href,
  logo: new URL('./assets/logo.png', import.meta.url).href,
  qr: new URL('./assets/qr.svg', import.meta.url).href,
  clover: new URL('./assets/clover.png', import.meta.url).href,
};


// ===================== CONFIG (mexa aqui) =====================
const CONFIG = {
  drawIntervalSec: 20,      // tempo entre um sorteio e o próximo. PRODUÇÃO: ex. 600 (10 min)
  spinMs: 700,              // quanto tempo cada bola "roda" antes de travar
  gapMs: 180,               // pausa entre travar uma bola e começar a próxima
  digits: 6,                // quantidade de dígitos sorteados
  drawTime: '10:00',        // HORÁRIO do sorteio, exibido na pílula (estático)
  luckySpeed: 0.5,          // velocidade da pose idle (1 = original; <1 mais lento)
  reactionSpeed: 0.75,      // velocidade das reações (0.5 atual x1,5)
  reactionEverySec: 5,      // dispara uma reação a cada X segundos
  nextDrawSec: 24*60 + 48,  // contador longo do card "próximo sorteio" (cosmético)
  nextDrawLabel: 'Hoje às 11h',
};
// =============================================================

const WD  = ['Domingo','Segunda-feira','Terça-feira','Quarta-feira','Quinta-feira','Sexta-feira','Sábado'];
const pad = (n) => String(n).padStart(2, '0');

const scale = ref(1);
const luckyEl = ref(null);   // container da Lottie do Lucky
function fit(){ scale.value = Math.min(window.innerWidth/1080, window.innerHeight/1920); }

const dateLabel = ref('');
const clock     = ref('');
function tickClock(){
  const d = new Date();
  dateLabel.value = WD[d.getDay()] + ', ' + pad(d.getDate()) + '/' + pad(d.getMonth()+1);
  clock.value     = pad(d.getHours()) + ':' + pad(d.getMinutes()) + ':' + pad(d.getSeconds());
}

const sorteioNum = ref(114);

// pílula: horário do sorteio (estático, só exibição)
const drawTime = ref(CONFIG.drawTime);
// gatilho interno que dispara cada novo sorteio (não é exibido)
const pillLeft = ref(CONFIG.drawIntervalSec);

// card: contador longo (só desce e reinicia — realismo)
const nextLeft = ref(CONFIG.nextDrawSec);
const nextH    = computed(() => pad(Math.floor(nextLeft.value/3600)));
const nextM    = computed(() => pad(Math.floor((nextLeft.value % 3600)/60)));
const nextS    = computed(() => pad(nextLeft.value % 60));
const nextLabel = ref(CONFIG.nextDrawLabel);
const progress  = computed(() => 1 - nextLeft.value / CONFIG.nextDrawSec);

// bolas
const balls = reactive(Array.from({ length: CONFIG.digits }, () => ({ digit:'', spinning:false, landed:false, glow:false })));
const numberStr = computed(() => balls.every(b => b.landed) ? balls.map(b => b.digit).join('') : '');
const complete  = computed(() => numberStr.value.length === CONFIG.digits);

// prêmios: sufixos do número (Sena=6, Quina=5, Quadra=4, Trinca=3, Dupla=2)
const TIER_DEF = [
  { name:'Sena',   icon:'💰', take:6 },
  { name:'Quina',  icon:'⭐', take:5 },
  { name:'Quadra', icon:'🍀', take:4 },
  { name:'Trinca', icon:'✨', take:3 },
  { name:'Dupla',  icon:'✌️', take:2 },
];
const tiers = computed(() => TIER_DEF.map(t => ({
  name:  t.name,
  icon:  t.icon,
  value: complete.value ? numberStr.value.slice(-t.take) : '••••••'.slice(0, t.take),
  show:  complete.value,
})));

// ganhadores (conta subindo)
const winnersDisplay = ref('0');
const winnersShown = computed(() => complete.value ? winnersDisplay.value : '•••');
function animateWinners(to){
  const dur = 1200, t0 = performance.now();
  function step(t){
    const k = Math.min(1, (t - t0) / dur);
    winnersDisplay.value = String(Math.round(to * (1 - Math.pow(1 - k, 3))));
    if (k < 1) requestAnimationFrame(step);
  }
  requestAnimationFrame(step);
}

// revelar uma bola: gira dígitos aleatórios e trava no final
function revealBall(i){
  return new Promise((resolve) => {
    const b = balls[i];
    b.spinning = true; b.landed = false; b.glow = false;
    const final = Math.floor(Math.random() * 10);
    const spin  = setInterval(() => { b.digit = String(Math.floor(Math.random() * 10)); }, 60);
    setTimeout(() => {
      clearInterval(spin);
      b.digit = String(final);
      b.spinning = false; b.landed = true; b.glow = true;
      setTimeout(() => { b.glow = false; }, 900);
      resolve();
    }, CONFIG.spinMs);
  });
}

let revealing = false;
async function drawNumber(){
  if (revealing) return;
  revealing = true;
  winnersDisplay.value = '0';
  balls.forEach(b => { b.digit=''; b.landed=false; b.spinning=false; b.glow=false; });
  for (let i = 0; i < CONFIG.digits; i++){
    await revealBall(i);
    await new Promise(r => setTimeout(r, CONFIG.gapMs));
  }
  revealing = false;
  animateWinners(40 + Math.floor(Math.random() * 300));
}

// números decorativos do fundo
const bgNums = [
  {t:'57',x:6, y:14,r:-16,s:70},{t:'58',x:66,y:9, r:12, s:66},{t:'54',x:3, y:34,r:-8, s:64},
  {t:'11',x:20,y:8, r:22, s:52},{t:'36',x:78,y:22,r:-14,s:60},{t:'30',x:40,y:6, r:8,  s:48},
  {t:'78',x:74,y:38,r:18, s:58},{t:'43',x:2, y:56,r:-20,s:66},{t:'41',x:14,y:62,r:14, s:56},
  {t:'39',x:34,y:60,r:-10,s:70},{t:'37',x:60,y:56,r:16, s:52},{t:'54',x:24,y:78,r:-12,s:60},
  {t:'11',x:84,y:60,r:10, s:48},{t:'62',x:50,y:44,r:-6, s:44},
];

onMounted(() => {
  fit();
  window.addEventListener('resize', fit);
  tickClock();
  drawNumber();
  setInterval(tickClock, 1000);
  setInterval(() => {
    if (pillLeft.value > 0) { pillLeft.value--; }
    else { sorteioNum.value++; pillLeft.value = CONFIG.drawIntervalSec; drawNumber(); }
    nextLeft.value = nextLeft.value > 0 ? nextLeft.value - 1 : CONFIG.nextDrawSec;
  }, 1000);

  // ---------- LUCKY (Lottie) ----------
  // Pose base em loop. Quando você mandar as reações, elas entram em LUCKY.reactions
  // e o mecanismo abaixo sorteia uma a cada X seg, toca 1x e volta pro idle.
  const LUCKY = {
    idle: luckyData,          // pose neutra (loop)
    reactions: luckyReactions,// poses one-shot (blink, coins, cool, fun, happy)
    minGap: CONFIG.reactionEverySec,
    maxGap: CONFIG.reactionEverySec,
  };
  let anim = lottie.loadAnimation({
    container: luckyEl.value, renderer:'svg', loop:true, autoplay:true, animationData: LUCKY.idle,
  });
  anim.setSpeed(CONFIG.luckySpeed);
  function playIdle(){
    anim.destroy();
    anim = lottie.loadAnimation({ container:luckyEl.value, renderer:'svg', loop:true, autoplay:true, animationData: LUCKY.idle });
    anim.setSpeed(CONFIG.luckySpeed);
  }
  function playReaction(){
    if (!LUCKY.reactions.length) { scheduleReaction(); return; }
    const data = LUCKY.reactions[Math.floor(Math.random()*LUCKY.reactions.length)];
    anim.destroy();
    anim = lottie.loadAnimation({ container:luckyEl.value, renderer:'svg', loop:false, autoplay:true, animationData:data });
    anim.setSpeed(CONFIG.reactionSpeed);
    anim.addEventListener('complete', () => { playIdle(); scheduleReaction(); });
  }
  function scheduleReaction(){
    const gap = (LUCKY.minGap + Math.random()*(LUCKY.maxGap-LUCKY.minGap)) * 1000;
    setTimeout(playReaction, gap);
  }
  scheduleReaction();
});

</script>

<style scoped>

@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@600;700;800;900&display=swap');

*{ box-sizing:border-box; margin:0; padding:0; }

.viewport{
  position:fixed; inset:0; overflow:hidden;
  background:#08021f;
  font-family:'Nunito', system-ui, -apple-system, sans-serif;
}
.scaler{
  position:absolute; left:50%; top:50%;
  width:1080px; height:1920px; transform-origin:center center;
}
.stage{
  position:relative; width:1080px; height:1920px; overflow:hidden;
  background:#0a0233;
}

/* ---------- fundo ---------- */
.bg-base{
  position:absolute; inset:0;
  background:radial-gradient(120% 85% at 68% 42%, #331074 0%, #1a075a 42%, #0a0233 100%);
}
.bg-cosmic{
  position:absolute; inset:0; width:100%; height:100%;
  object-fit:cover; opacity:.5; mix-blend-mode:screen;
}
.bg-numbers span{
  position:absolute; font-weight:800; color:#7b46e0; opacity:.12;
  text-shadow:0 0 20px rgba(123,70,224,.4); user-select:none;
}
.lucky{
  position:absolute; right:-206px; bottom:0; width:1081px; height:1560px; z-index:25;
  pointer-events:none;
}
.lucky svg, .lucky canvas{ display:block; width:100% !important; height:100% !important; }
.halo{
  position:absolute; left:0; bottom:-40px; width:100%; z-index:1;
  pointer-events:none; transform-origin:62% 72%;
  animation:haloBreath 9s ease-in-out infinite;
}
.coin{ position:absolute; z-index:7; pointer-events:none; filter:drop-shadow(0 6px 10px rgba(0,0,0,.35)); }
.c1{ left:288px; bottom:430px; width:130px; animation:cSpin   7.0s ease-in-out infinite; }
.c2{ left:250px; bottom:250px; width:88px;  animation:cTumble 5.6s ease-in-out infinite .8s; }
.c3{ left:392px; bottom:552px; width:96px;  animation:cBob    6.2s ease-in-out infinite .3s; }
.c4{ left:330px; bottom:178px; width:92px;  animation:cTumble 8.0s linear      infinite 1.2s reverse; }
.c5{ left:452px; bottom:372px; width:66px;  animation:cSpin   5.0s ease-in-out infinite .5s; }
.c6{ left:540px; bottom:130px; width:76px;  animation:cBob    6.8s ease-in-out infinite 1.8s; }
.c7{ left:250px; bottom:600px; width:70px;  animation:cSpin   5.8s ease-in-out infinite 2.2s reverse; }
.c8{ left:486px; bottom:486px; width:52px;  animation:cTumble 4.6s linear      infinite 1.0s; }
@keyframes cSpin{
  0%  { transform:translateY(0)     rotate(0deg); }
  50% { transform:translateY(-26px) rotate(180deg); }
  100%{ transform:translateY(0)     rotate(360deg); }
}
@keyframes cTumble{
  0%  { transform:translate(0,0)     rotate(0deg); }
  50% { transform:translate(8px,-30px) rotate(190deg); }
  100%{ transform:translate(0,0)     rotate(360deg); }
}
@keyframes cBob{
  0%,100%{ transform:translateY(0)   rotate(-10deg); }
  50%    { transform:translateY(-22px) rotate(10deg); }
}
@keyframes haloBreath{
  0%,100%{ transform:scale(1) rotate(-2deg); opacity:.8; }
  50%    { transform:scale(1.05) rotate(2deg); opacity:.96; }
}

/* ---------- header ---------- */
.hdr{
  position:absolute; left:0; top:0; width:1080px; height:118px;
  background:#fff; z-index:20; box-shadow:0 2px 0 rgba(0,0,0,.06);
}
.logo{ position:absolute; left:40px; top:50%; transform:translateY(-50%); height:74px; }
.hdr-right{ position:absolute; right:44px; top:12px; text-align:right; }
.date{ color:#7C3E96; font-weight:800; font-size:26px; }
.clock{ color:#1b1330; font-weight:800; font-size:44px; line-height:1; font-variant-numeric:tabular-nums; letter-spacing:1px; }

/* ---------- pílula ---------- */
@property --pa{ syntax:'<angle>'; initial-value:0deg; inherits:false; }
.pill{
  position:absolute; left:60px; top:184px; width:960px; height:86px;
  border-radius:44px; padding:0 42px; z-index:15;
  border:3px solid transparent;
  background:
    linear-gradient(#2c0d59,#210a40) padding-box,
    conic-gradient(from var(--pa), #6D406E 0deg, #ECA647 86deg, #F8D92A 176deg, #D5944E 302deg, #6D406E 360deg) border-box;
  display:flex; align-items:center; justify-content:space-between;
  color:#fff; font-weight:800; font-size:42px;
  box-shadow:0 8px 30px rgba(0,0,0,.4), 0 0 34px rgba(246,197,63,.28);
  animation:pillspin 6s linear infinite;
}
@keyframes pillspin{ to{ --pa:360deg } }
.pill-left{ display:flex; align-items:center; }
.pill .clover{ height:52px; width:auto; margin-right:16px; filter:drop-shadow(0 2px 6px rgba(0,0,0,.35)); }
.pill-right{ display:flex; align-items:center; gap:14px; color:#DEE8FB; font-weight:600; }
.pill-right svg{ width:38px; height:38px; stroke:#D8BBE4; }

/* ---------- título ---------- */
.title{
  position:absolute; top:298px; left:0; width:1080px; z-index:15;
  display:flex; align-items:center; justify-content:center; gap:22px;
  color:#f6c53f; font-weight:900; font-size:48px; letter-spacing:5px;
  text-shadow:0 0 20px rgba(246,197,63,.35);
}
.title-text{ white-space:nowrap; }
.flourish{ position:relative; width:130px; height:3px; flex:none; }
.flourish.left{ background:linear-gradient(90deg, transparent, #f6c53f); }
.flourish.right{ background:linear-gradient(270deg, transparent, #f6c53f); }
.flourish::after{
  content:''; position:absolute; top:50%; width:12px; height:12px;
  background:#f6c53f; transform:translateY(-50%) rotate(45deg);
}
.flourish.left::after{ right:-4px; }
.flourish.right::after{ left:-4px; }

/* ---------- bolas ---------- */
.balls{
  position:absolute; left:50px; top:398px; width:980px; height:158px;
  display:flex; justify-content:space-between; z-index:15;
}
.ball{
  width:150px; height:150px; border-radius:50%; position:relative;
  display:flex; align-items:center; justify-content:center;
  background:radial-gradient(circle at 38% 30%, #ffffff 0%, #f3e7f4 55%, #e0cee8 100%);
  box-shadow:0 0 0 3px rgba(200,172,232,.55),
             0 0 26px 6px rgba(246,197,63,.40),
             0 12px 26px rgba(0,0,0,.4),
             inset 0 -9px 16px rgba(150,120,160,.4),
             inset 0 9px 14px rgba(255,255,255,.95);
  transition:box-shadow .3s ease;
}
.ball .digit{
  font-weight:900; font-size:94px; line-height:1; color:#0e1a5b;
  font-variant-numeric:tabular-nums;
}
.ball.spin .digit{ filter:blur(1.2px); opacity:.8; }
.ball.land{ animation:pop .45s cubic-bezier(.2,1.5,.4,1); }
.ball.glow{
  box-shadow:0 0 0 5px rgba(246,197,63,.95),
             0 0 46px 12px rgba(246,197,63,.78),
             0 12px 26px rgba(0,0,0,.45),
             inset 0 -9px 16px rgba(150,120,160,.4),
             inset 0 9px 14px rgba(255,255,255,.95);
}

/* ---------- prêmios ---------- */
.tiers{
  position:absolute; left:50px; top:612px; width:980px; height:258px; z-index:15;
  border-radius:30px; display:flex;
  background:rgba(30,10,66,.6); border:1px solid rgba(246,197,63,.22);
  box-shadow:inset 0 1px 0 rgba(255,255,255,.06), 0 12px 34px rgba(0,0,0,.32);
}
.tier{ flex:1; display:flex; flex-direction:column; align-items:center; justify-content:center; gap:10px; position:relative; }
.tier + .tier::before{
  content:''; position:absolute; left:0; top:26%; height:48%; width:1px; background:rgba(255,255,255,.10);
}
.tier-icon{ font-size:58px; line-height:1; }
.tier-name{ color:#fff; font-weight:800; font-size:38px; }
.tier-value{
  color:#f6c53f; font-weight:600; font-size:42px; letter-spacing:2px;
  font-variant-numeric:tabular-nums; min-height:50px; opacity:.28; transform:translateY(6px);
}
.tier.show .tier-value{ animation:rise .5s ease both; animation-delay:var(--d); }

/* ---------- ganhadores ---------- */
.winners{
  position:absolute; left:50px; top:892px; width:980px; height:104px; z-index:15;
  border-radius:24px; display:flex; align-items:center; gap:22px; padding:0 32px;
  background:rgba(30,10,66,.6); border:1px solid rgba(246,197,63,.16);
  box-shadow:0 12px 30px rgba(0,0,0,.28);
}
.trophy{ height:98px; }
.winners-txt{ color:#fff; font-weight:600; font-size:44px; }
.winners-txt b{ color:#f6c53f; font-weight:800; font-size:50px; }
.winners-txt b.dim{ color:#f6c53f; font-weight:600; opacity:.28; letter-spacing:3px; }

/* ---------- próximo sorteio ---------- */
.card{
  position:absolute; left:60px; top:1058px; width:440px; height:300px; z-index:15;
  border-radius:30px; background:#fff; padding:30px 36px;
  box-shadow:0 22px 55px rgba(0,0,0,.5);
}
.card-kicker{ color:#7C3E96; font-weight:800; font-size:26px; letter-spacing:2px; }
.card-when{ color:#241a3d; font-weight:900; font-size:48px; line-height:1.05; margin-top:6px; white-space:nowrap; }
.cd{ display:flex; align-items:flex-start; gap:6px; margin-top:16px; }
.cd-grp{ display:flex; flex-direction:column; align-items:center; width:98px; }
.cd-num{ color:#7C3E96; font-weight:900; font-size:82px; line-height:.9; font-variant-numeric:tabular-nums; }
.cd-lab{ color:#8a84a0; font-weight:700; font-size:26px; margin-top:8px; }
.cd-sep{ color:#cdc6dc; font-weight:900; font-size:66px; line-height:1; padding-top:4px; }
.bar{ position:absolute; left:36px; right:36px; bottom:26px; height:9px; border-radius:6px; background:#ece7f5; overflow:hidden; }
.bar-fill{ height:100%; border-radius:6px; background:linear-gradient(90deg,#a45fd0,#7C3E96); transition:width 1s linear; }

/* ---------- QR ---------- */
.qr{ position:absolute; left:60px; top:1556px; width:300px; z-index:30; }

/* ---------- animações ---------- */
@keyframes breathe{ 0%,100%{ transform:translateY(0); } 50%{ transform:translateY(-8px); } }
@keyframes pop{ 0%{ transform:scale(.5); } 60%{ transform:scale(1.12); } 100%{ transform:scale(1); } }
@keyframes rise{ to{ opacity:1; transform:none; } }

</style>
