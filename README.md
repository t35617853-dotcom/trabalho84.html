<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Slideshow</title>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      background: #060608;
      width: 100vw;
      height: 100vh;
      overflow: hidden;
      font-family: 'Inter', 'Segoe UI', sans-serif;
      color: #fff;
    }

    #bg-layer {
      position: fixed;
      inset: 0;
      background-size: cover;
      background-position: center;
      opacity: 0.09;
      z-index: 0;
      transition: background-image 1s ease, opacity 1s ease;
      pointer-events: none;
    }

    .orb {
      position: fixed;
      border-radius: 50%;
      filter: blur(60px);
      pointer-events: none;
      z-index: 1;
      animation: orbFloat linear infinite;
    }
    @keyframes orbFloat {
      0%   { transform: translate(0,0) scale(1); }
      25%  { transform: translate(30px,-20px) scale(1.05); }
      50%  { transform: translate(-15px,25px) scale(0.95); }
      75%  { transform: translate(20px,10px) scale(1.03); }
      100% { transform: translate(0,0) scale(1); }
    }

    .dot {
      position: fixed;
      border-radius: 50%;
      pointer-events: none;
      z-index: 1;
      animation: dotFloat ease-in-out infinite;
      opacity: 0;
    }
    @keyframes dotFloat {
      0%   { transform: translateY(0) scale(0); opacity: 0; }
      20%  { opacity: 0.5; transform: translateY(-20px) scale(1); }
      80%  { opacity: 0.3; }
      100% { transform: translateY(-80px) scale(0); opacity: 0; }
    }

    #vignette {
      position: fixed;
      inset: 0;
      background: radial-gradient(ellipse at center, transparent 38%, rgba(0,0,0,0.78) 100%);
      z-index: 2;
      pointer-events: none;
      animation: vignettePulse 4s ease-in-out infinite;
    }
    @keyframes vignettePulse {
      0%, 100% { opacity: 0.75; }
      50%       { opacity: 0.9; }
    }

    #stage {
      position: fixed;
      inset: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 4;
      perspective: 1200px;
    }

    .slide {
      position: absolute;
      inset: 0;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .slide img {
      max-width: 88vw;
      max-height: 82vh;
      object-fit: contain;
      border-radius: 12px;
      display: block;
      user-select: none;
      pointer-events: none;
    }

    @keyframes kb0 { from { transform: scale(1.08) translate(0%,0%); }    to { transform: scale(1.02) translate(-2%,-1%); } }
    @keyframes kb1 { from { transform: scale(1.05) translate(-1%,-1%); }  to { transform: scale(1.10) translate(1%,1%); } }
    @keyframes kb2 { from { transform: scale(1.10) translate(2%,1%); }    to { transform: scale(1.04) translate(-1%,-1%); } }
    @keyframes kb3 { from { transform: scale(1.04) translate(-2%,0%); }   to { transform: scale(1.09) translate(0%,2%); } }
    .kb0 { animation: kb0 8s linear forwards; }
    .kb1 { animation: kb1 8s linear forwards; }
    .kb2 { animation: kb2 8s linear forwards; }
    .kb3 { animation: kb3 8s linear forwards; }

    #glow-frame {
      position: fixed;
      inset: 5vh 5vw;
      border-radius: 16px;
      z-index: 3;
      pointer-events: none;
      transition: box-shadow 1.2s ease;
    }

    #shimmer {
      position: fixed;
      inset: 0;
      background: linear-gradient(105deg, transparent 20%, rgba(255,255,255,0.10) 45%, rgba(255,255,255,0.22) 50%, rgba(255,255,255,0.10) 55%, transparent 80%);
      z-index: 7;
      pointer-events: none;
      transform: translateX(-120%);
    }
    #shimmer.sweep {
      animation: shimmerSweep 0.9s cubic-bezier(0.22,1,0.36,1) forwards;
    }
    @keyframes shimmerSweep {
      from { transform: translateX(-120%); opacity: 0.8; }
      to   { transform: translateX(220%);  opacity: 0; }
    }

    .enter-fade-zoom  { animation: enterFadeZoom  0.8s cubic-bezier(0.22,1,0.36,1) forwards; }
    .exit-fade-zoom   { animation: exitFadeZoom   0.8s cubic-bezier(0.22,1,0.36,1) forwards; }
    @keyframes enterFadeZoom { from { opacity:0; transform:scale(1.15); } to { opacity:1; transform:scale(1); } }
    @keyframes exitFadeZoom  { from { opacity:1; transform:scale(1); }  to { opacity:0; transform:scale(0.88); } }

    .enter-slide-r    { animation: enterSlideR    0.75s cubic-bezier(0.25,0.46,0.45,0.94) forwards; }
    .exit-slide-r     { animation: exitSlideR     0.75s cubic-bezier(0.25,0.46,0.45,0.94) forwards; }
    @keyframes enterSlideR { from { opacity:0.5; transform:translateX(100%) scale(0.92); } to { opacity:1; transform:translateX(0) scale(1); } }
    @keyframes exitSlideR  { from { opacity:1; transform:translateX(0); }  to { opacity:0; transform:translateX(-40%) scale(0.88); } }

    .enter-slide-l    { animation: enterSlideL    0.75s cubic-bezier(0.25,0.46,0.45,0.94) forwards; }
    .exit-slide-l     { animation: exitSlideL     0.75s cubic-bezier(0.25,0.46,0.45,0.94) forwards; }
    @keyframes enterSlideL { from { opacity:0.5; transform:translateX(-100%) scale(0.92); } to { opacity:1; transform:translateX(0) scale(1); } }
    @keyframes exitSlideL  { from { opacity:1; transform:translateX(0); }  to { opacity:0; transform:translateX(40%) scale(0.88); } }

    .enter-vortex     { animation: enterVortex    0.85s cubic-bezier(0.22,1,0.36,1) forwards; }
    .exit-vortex      { animation: exitVortex     0.85s cubic-bezier(0.22,1,0.36,1) forwards; }
    @keyframes enterVortex { from { opacity:0; transform:scale(0) rotate(270deg); filter:blur(20px); } to { opacity:1; transform:scale(1) rotate(0deg); filter:blur(0); } }
    @keyframes exitVortex  { from { opacity:1; transform:scale(1) rotate(0deg); } to { opacity:0; transform:scale(0) rotate(-180deg); filter:blur(20px); } }

    .enter-flip       { animation: enterFlip      0.7s cubic-bezier(0.25,0.46,0.45,0.94) forwards; }
    .exit-flip        { animation: exitFlip       0.7s cubic-bezier(0.25,0.46,0.45,0.94) forwards; }
    @keyframes enterFlip { from { opacity:0; transform:rotateY(90deg) scale(0.8); } to { opacity:1; transform:rotateY(0deg) scale(1); } }
    @keyframes exitFlip  { from { opacity:1; transform:rotateY(0deg) scale(1); }  to { opacity:0; transform:rotateY(-90deg) scale(0.8); } }

    .enter-spiral     { animation: enterSpiral    0.9s cubic-bezier(0.22,1,0.36,1) forwards; }
    .exit-spiral      { animation: exitSpiral     0.9s cubic-bezier(0.22,1,0.36,1) forwards; }
    @keyframes enterSpiral { from { opacity:0; transform:scale(0.2) rotate(360deg); filter:blur(8px); } to { opacity:1; transform:scale(1) rotate(0deg); filter:blur(0); } }
    @keyframes exitSpiral  { from { opacity:1; transform:scale(1) rotate(0deg); }  to { opacity:0; transform:scale(0) rotate(-180deg); filter:blur(8px); } }

    .enter-curtain    { animation: enterCurtain   0.7s cubic-bezier(0.77,0,0.175,1) forwards; transform-origin: left center; }
    .exit-curtain     { animation: exitCurtain    0.7s cubic-bezier(0.77,0,0.175,1) forwards; transform-origin: right center; }
    @keyframes enterCurtain { from { transform:scaleX(0); } to { transform:scaleX(1); } }
    @keyframes exitCurtain  { from { transform:scaleX(1); } to { transform:scaleX(0); } }

    .enter-cube       { animation: enterCube      0.75s cubic-bezier(0.25,0.46,0.45,0.94) forwards; }
    .exit-cube        { animation: exitCube       0.75s cubic-bezier(0.25,0.46,0.45,0.94) forwards; }
    @keyframes enterCube { from { opacity:0; transform:rotateY(90deg) translateX(30%) scale(0.75); } to { opacity:1; transform:rotateY(0deg) translateX(0) scale(1); } }
    @keyframes exitCube  { from { opacity:1; transform:rotateY(0deg) translateX(0) scale(1); }  to { opacity:0; transform:rotateY(-90deg) translateX(-30%) scale(0.75); } }

    .enter-dissolve   { animation: enterDissolve  0.9s ease-in-out forwards; }
    .exit-dissolve    { animation: exitDissolve   0.9s ease-in-out forwards; }
    @keyframes enterDissolve { from { opacity:0; transform:scale(1.12); filter:blur(40px) saturate(2); } to { opacity:1; transform:scale(1); filter:blur(0) saturate(1); } }
    @keyframes exitDissolve  { from { opacity:1; transform:scale(1); filter:blur(0); } to { opacity:0; transform:scale(0.9); filter:blur(40px) saturate(0); } }

    .enter-morph      { animation: enterMorph     0.8s cubic-bezier(0.77,0,0.175,1) forwards; }
    .exit-morph       { animation: exitMorph      0.8s cubic-bezier(0.77,0,0.175,1) forwards; }
    @keyframes enterMorph { from { clip-path:inset(50% 50% 50% 50% round 50%); opacity:0.8; } to { clip-path:inset(0% 0% 0% 0% round 0px); opacity:1; } }
    @keyframes exitMorph  { from { clip-path:inset(0% 0% 0% 0% round 0px); opacity:1; } to { clip-path:inset(50% 50% 50% 50% round 50%); opacity:0; } }

    .enter-glitch     { animation: enterGlitch    0.6s cubic-bezier(0.25,0.46,0.45,0.94) forwards; }
    .exit-glitch      { animation: exitGlitch     0.6s cubic-bezier(0.25,0.46,0.45,0.94) forwards; }
    @keyframes enterGlitch { from { opacity:0; transform:translateX(30px) scale(1.05); filter:hue-rotate(90deg) contrast(2); } to { opacity:1; transform:translateX(0) scale(1); filter:hue-rotate(0deg) contrast(1); } }
    @keyframes exitGlitch  { from { opacity:1; transform:translateX(0) scale(1); filter:hue-rotate(0deg) contrast(1); } to { opacity:0; transform:translateX(-30px) scale(0.95); filter:hue-rotate(-90deg) contrast(2); } }

    #controls {
      position: fixed;
      inset: 0;
      z-index: 10;
      pointer-events: none;
      transition: opacity 0.4s ease;
    }
    #controls.visible { pointer-events: auto; opacity: 1; }
    #controls.hidden  { pointer-events: none; opacity: 0; }

    .arrow {
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      width: 56px; height: 56px;
      border-radius: 50%;
      background: rgba(255,255,255,0.06);
      backdrop-filter: blur(16px);
      border: 1px solid rgba(255,255,255,0.10);
      color: #fff;
      cursor: pointer;
      display: flex; align-items: center; justify-content: center;
      transition: background 0.25s, box-shadow 0.25s, transform 0.25s;
    }
    .arrow:hover {
      background: rgba(255,255,255,0.18);
      box-shadow: 0 0 30px rgba(129,140,248,0.35);
      border-color: rgba(129,140,248,0.5);
      transform: translateY(-50%) scale(1.1);
    }
    .arrow:active { transform: translateY(-50%) scale(0.93); }
    #btn-prev { left: 20px; }
    #btn-next { right: 20px; }

    #bottom {
      position: absolute;
      bottom: 28px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 14px;
    }

    #dots { display: flex; gap: 8px; align-items: center; }
    .dot-btn {
      height: 10px;
      border-radius: 5px;
      border: none;
      background: rgba(255,255,255,0.22);
      cursor: pointer;
      transition: width 0.4s cubic-bezier(0.25,0.46,0.45,0.94), background 0.4s, box-shadow 0.4s;
      width: 10px;
    }
    .dot-btn.active { width: 32px; background: rgba(255,255,255,0.95); box-shadow: 0 0 12px rgba(192,132,252,0.6); }
    .dot-btn.past { background: rgba(129,140,248,0.5); }
    #info-bar {
      display: flex;
      align-items: center;
      gap: 14px;
      padding: 8px 22px;
      border-radius: 40px;
      background: rgba(0,0,0,0.55);
      backdrop-filter: blur(16px);
      border: 1px solid rgba(255,255,255,0.08);
      box-shadow: 0 4px 24px rgba(0,0,0,0.4);
      white-space: nowrap;
    }
    .sep { width:1px; height:16px; background:rgba(255,255,255,0.12); flex-shrink:0; }
    #counter { font-size:13px; font-weight:600; letter-spacing:0.08em; color:rgba(255,255,255,0.85); min-width:48px; text-align:center; }
    #btn-auto { display:flex; align-items:center; gap:6px; background:none; border:none; color:rgba(255,255,255,0.45); cursor:pointer; font-size:12px; font-weight:600; letter-spacing:0.06em; text-transform:uppercase; transition:color 0.3s; }
    #btn-auto.playing { color: #a78bfa; }
    #btn-fs { background:none; border:none; color:rgba(255,255,255,0.4); cursor:pointer; display:flex; align-items:center; transition:color 0.3s; }
    #btn-fs:hover, #btn-fs.active { color: #a78bfa; }
    #transition-label { color:rgba(167,139,250,0.7); font-size:10px; font-weight:600; text-transform:uppercase; letter-spacing:0.12em; min-width:70px; }
    #progress-track {
      position: fixed;
      top: 20px; left: 50%; transform: translateX(-50%);
      width: 240px; height: 3px;
      background: rgba(255,255,255,0.07);
      border-radius: 3px; overflow: visible;
      z-index: 11;
    }
    #progress-fill {
      height: 100%; border-radius: 3px;
      background: linear-gradient(90deg, #6366f1, #a78bfa, #ec4899);
      box-shadow: 0 0 10px rgba(167,139,250,0.5);
      transition: width 0.55s cubic-bezier(0.25,0.46,0.45,0.94);
      position: relative;
    }
    #progress-dot {
      position: absolute;
      right: -1px; top: -2px;
      width: 7px; height: 7px;
      border-radius: 50%;
      background: #ec4899;
      box-shadow: 0 0 10px #ec4899;
      animation: dotPulse 1.5s ease-in-out infinite;
    }
    @keyframes dotPulse { 0%,100%{opacity:0.5;} 50%{opacity:1;} }
    #timer-bar {
      position: fixed;
      bottom: 0; left: 0;
      height: 3px;
      background: linear-gradient(90deg,#6366f1,#a78bfa,#ec4899);
      box-shadow: 0 0 8px rgba(167,139,250,0.6);
      z-index: 20;
      width: 0;
      display: none;
    }
    #timer-bar.running { display: block; animation: timerFill 4.5s linear forwards; }
    @keyframes timerFill { from{width:0%} to{width:100%} }
    #hint {
      position: fixed;
      bottom: 100px; left: 50%; transform: translateX(-50%);
      background: rgba(0,0,0,0.6);
      backdrop-filter: blur(10px);
      border: 1px solid rgba(255,255,255,0.08);
      padding: 6px 16px;
      border-radius: 20px;
      font-size: 11px;
      color: rgba(255,255,255,0.4);
      letter-spacing: 0.06em;
      z-index: 12;
      pointer-events: none;
      opacity: 0;
      transition: opacity 0.5s;
    }
    #hint.show { opacity: 1; }
  </style>
</head>
<body>
  <div id="bg-layer"></div>
  <div class="orb" style="width:600px;height:600px;left:10%;top:20%;background:radial-gradient(circle,rgba(99,102,241,0.12),transparent 70%);animation-duration:18s;"></div>
  <div class="orb" style="width:500px;height:500px;left:80%;top:70%;background:radial-gradient(circle,rgba(167,139,250,0.10),transparent 70%);animation-duration:22s;"></div>
  <div class="orb" style="width:400px;height:400px;left:60%;top:10%;background:radial-gradient(circle,rgba(236,72,153,0.08),transparent 70%);animation-duration:15s;"></div>
  <div class="orb" style="width:350px;height:350px;left:20%;top:80%;background:radial-gradient(circle,rgba(251,191,36,0.06),transparent 70%);animation-duration:26s;"></div>
  <div class="orb" style="width:450px;height:450px;left:50%;top:50%;background:radial-gradient(circle,rgba(129,140,248,0.07),transparent 70%);animation-duration:20s;"></div>
  <div id="vignette"></div>
  <div id="shimmer"></div>
  <div id="glow-frame"></div>
  <div id="stage"></div>
  <div id="progress-track">
    <div id="progress-fill"><div id="progress-dot"></div></div>
  </div>
  <div id="controls" class="visible">
    <button class="arrow" id="btn-prev">
      <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="15 18 9 12 15 6"/></svg>
    </button>
    <button class="arrow" id="btn-next">
      <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="9 18 15 12 9 6"/></svg>
    </button>
    <div id="bottom">
      <div id="dots"></div>
      <div id="info-bar">
        <span id="counter">01 / 09</span>
        <div class="sep"></div>
        <button id="btn-auto">
          <svg id="icon-auto" width="12" height="12" viewBox="0 0 24 24" fill="currentColor"><polygon points="5 3 19 12 5 21"/></svg>
          <span id="label-auto">Auto</span>
        </button>
        <div class="sep"></div>
        <button id="btn-fs" title="Tela cheia (F)">
          <svg id="icon-fs-expand" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="15 3 21 3 21 9"/><polyline points="9 21 3 21 3 15"/><line x1="21" y1="3" x2="14" y2="10"/><line x1="3" y1="21" x2="10" y2="14"/></svg>
          <svg id="icon-fs-compress" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" style="display:none"><polyline points="8 3 3 3 3 8"/><polyline points="21 8 21 3 16 3"/><polyline points="3 16 3 21 8 21"/><polyline points="16 21 21 21 21 16"/></svg>
        </button>
        <div class="sep"></div>
        <span id="transition-label">Fade Zoom</span>
      </div>
    </div>
  </div>
  <div id="timer-bar"></div>
  <div id="hint">← → Navegar &nbsp;|&nbsp; Espaço Avançar &nbsp;|&nbsp; F Tela cheia</div>
  <script>
    const SLIDES = [
      "https://i.imgur.com/ZscbHXs.jpeg",
      "https://i.imgur.com/enMulxB.jpeg",
      "https://i.imgur.com/8Tp8n6n.jpeg",
      "https://i.imgur.com/RkPg44N.jpeg",
      "https://i.imgur.com/t8PSurh.jpeg",
      "https://i.imgur.com/ihFdMVh.jpeg",
      "https://i.imgur.com/Dnxwj1m.jpeg",
      "https://i.imgur.com/jyHSUjG.jpeg",
      "https://i.imgur.com/REyVncF.png",
    ];
    const BG_IMAGES = [
      "https://i.imgur.com/eQauEJ7.jpeg",
      "https://i.imgur.com/SXOBnBE.jpeg",
    ];
    const TRANSITIONS = [
      { name: "Fade Zoom", enter: "enter-fade-zoom", exit: "exit-fade-zoom" },
      { name: "Slide",     enter: "enter-slide-r",   exit: "exit-slide-r"   },
      { name: "Vortex",    enter: "enter-vortex",    exit: "exit-vortex"    },
      { name: "Flip 3D",   enter: "enter-flip",      exit: "exit-flip"      },
      { name: "Spiral",    enter: "enter-spiral",    exit: "exit-spiral"    },
      { name: "Curtain",   enter: "enter-curtain",   exit: "exit-curtain"   },
      { name: "Cube 3D",   enter: "enter-cube",      exit: "exit-cube"      },
      { name: "Dissolve",  enter: "enter-dissolve",  exit: "exit-dissolve"  },
      { name: "Morph",     enter: "enter-morph",     exit: "exit-morph"     },
    ];
    const GLOWS = [
      "hsla(220,70%,55%,0.12)","hsla(260,70%,55%,0.12)","hsla(300,60%,50%,0.12)",
      "hsla(340,70%,55%,0.12)","hsla(40,80%,55%,0.10)","hsla(180,60%,45%,0.10)",
      "hsla(100,60%,45%,0.10)","hsla(200,70%,50%,0.12)","hsla(20,80%,55%,0.10)",
    ];
    let current = 0, busy = false, autoPlay = false, autoTimer = null, hideTimer = null;
    const stage        = document.getElementById("stage");
    const bgLayer      = document.getElementById("bg-layer");
    const glowFrame    = document.getElementById("glow-frame");
    const progressFill = document.getElementById("progress-fill");
    const counter      = document.getElementById("counter");
    const dotsEl       = document.getElementById("dots");
    const tlabel       = document.getElementById("transition-label");
    const controls     = document.getElementById("controls");
    const timerBar     = document.getElementById("timer-bar");
    const shimmer      = document.getElementById("shimmer");
    const hintEl       = document.getElementById("hint");
    const btnAuto      = document.getElementById("btn-auto");
    const btnFS        = document.getElementById("btn-fs");
    const iconAutoEl   = document.getElementById("icon-auto");
    const labelAuto    = document.getElementById("label-auto");
    const fsExpand     = document.getElementById("icon-fs-expand");
    const fsCompress   = document.getElementById("icon-fs-compress");
    SLIDES.forEach((_, i) => {
      const b = document.createElement("button");
      b.className = "dot-btn" + (i === 0 ? " active" : "");
      b.addEventListener("click", () => goTo(i));
      dotsEl.appendChild(b);
    });
    for (let i = 0; i < 20; i++) {
      const d = document.createElement("div");
      d.className = "dot";
      const size = 2 + Math.random() * 4;
      d.style.cssText = `width:${size}px;height:${size}px;left:${Math.random()*100}%;top:${Math.random()*100}%;background:hsla(${(i*60)%360},70%,70%,0.4);animation-duration:${3+Math.random()*4}s;animation-delay:${Math.random()*5}s;`;
      document.body.appendChild(d);
    }
    createSlide(0, null);
    updateUI();
    hintEl.classList.add("show");
    setTimeout(() => hintEl.classList.remove("show"), 3000);
    function createSlide(index, enterClass) {
      const el = document.createElement("div");
      el.className = "slide";
      const img = document.createElement("img");
      img.src = SLIDES[index];
      img.alt = "Slide " + (index + 1);
      img.className = "kb" + (index % 4);
      el.appendChild(img);
      if (enterClass) el.classList.add(enterClass);
      stage.appendChild(el);
      return el;
    }
    function triggerShimmer() {
      shimmer.classList.remove("sweep");
      void shimmer.offsetWidth;
      shimmer.classList.add("sweep");
    }
    function updateUI() {
      counter.textContent = String(current+1).padStart(2,"0") + " / " + String(SLIDES.length).padStart(2,"0");
      progressFill.style.width = ((current+1)/SLIDES.length*100) + "%";
      dotsEl.querySelectorAll(".dot-btn").forEach((b,i) => {
        b.className = "dot-btn" + (i===current?" active":i<current?" past":"");
      });
      glowFrame.style.boxShadow = `0 0 80px 20px ${GLOWS[current]}, 0 0 200px 60px ${GLOWS[current].replace("0.12","0.06").replace("0.10","0.05")}`;
      bgLayer.style.backgroundImage = `url(${BG_IMAGES[current%2]})`;
      tlabel.textContent = TRANSITIONS[current].name;
    }
    function goTo(index) {
      if (busy || index === current) return;
      busy = true;
      const t = TRANSITIONS[current];
      triggerShimmer();
      const oldSlide = stage.querySelector(".slide");
      if (oldSlide) {
        oldSlide.classList.add(t.exit);
        oldSlide.addEventListener("animationend", () => oldSlide.remove(), { once: true });
      }
      current = index;
      createSlide(current, TRANSITIONS[current].enter);
      updateUI();
      setTimeout(() => { busy = false; }, 900);
      if (autoPlay) resetTimerBar();
    }
    function next() { goTo((current+1) % SLIDES.length); }
    function prev() { goTo((current-1+SLIDES.length) % SLIDES.length); }
    function resetTimerBar() {
      timerBar.classList.remove("running");
      void timerBar.offsetWidth;
      timerBar.classList.add("running");
    }
    function startAuto() {
      autoPlay = true;
      btnAuto.classList.add("playing");
      iconAutoEl.innerHTML = '<rect x="6" y="4" width="4" height="16" rx="1.5"/><rect x="14" y="4" width="4" height="16" rx="1.5"/>';
      labelAuto.textContent = "Pause";
      timerBar.classList.add("running");
      autoTimer = setInterval(next, 4500);
    }
    function stopAuto() {
      autoPlay = false;
      btnAuto.classList.remove("playing");
      iconAutoEl.innerHTML = '<polygon points="5 3 19 12 5 21"/>';
      labelAuto.textContent = "Auto";
      timerBar.classList.remove("running");
      timerBar.style.display="none"; void timerBar.offsetWidth; timerBar.style.display="";
      clearInterval(autoTimer);
    }
    btnAuto.addEventListener("click", () => autoPlay ? stopAuto() : startAuto());
    function toggleFS() {
      if (!document.fullscreenElement) document.documentElement.requestFullscreen();
      else document.exitFullscreen();
    }
    btnFS.addEventListener("click", toggleFS);
    document.addEventListener("fullscreenchange", () => {
      const fs = !!document.fullscreenElement;
      fsExpand.style.display = fs ? "none" : "";
      fsCompress.style.display = fs ? "" : "none";
      btnFS.classList.toggle("active", fs);
    });
    document.getElementById("btn-prev").addEventListener("click", prev);
    document.getElementById("btn-next").addEventListener("click", next);
    document.addEventListener("keydown", e => {
      if (e.key==="ArrowRight"||e.key===" ") { e.preventDefault(); next(); }
      else if (e.key==="ArrowLeft") { e.preventDefault(); prev(); }
      else if (e.key==="f"||e.key==="F") toggleFS();
    });
    function resetHide() {
      controls.classList.remove("hidden"); controls.classList.add("visible");
      clearTimeout(hideTimer);
      hideTimer = setTimeout(() => { controls.classList.remove("visible"); controls.classList.add("hidden"); }, 3000);
    }
    document.addEventListener("mousemove", resetHide);
    document.addEventListener("touchstart", resetHide);
    resetHide();
    let touchX = 0;
    document.addEventListener("touchstart", e => { touchX = e.touches[0].clientX; }, { passive: true });
    document.addEventListener("touchend", e => {
      const dx = e.changedTouches[0].clientX - touchX;
      if (Math.abs(dx) > 50) dx < 0 ? next() : prev();
    });
  </script>
</body>
</html>
