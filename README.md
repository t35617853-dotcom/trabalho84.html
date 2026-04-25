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
    .dot
