<script>
  import { onMount } from 'svelte';
  import WhoIAm from './WhoIAm.svelte';
  import CircleSelectionPage from './CircleSelectionPage.svelte';
  import Work from "./Work.svelte";
  import renukaPhoto from '../assets/wbst 1.png';

  let mouseX = 0, mouseY = 0;
  let renukaRef, joagRef, curiousRef, optimisticRef, observantRef, experimentalRef;
  let returningFromDeep = false;
  let keepCircleVisible = false;
  let expandedRenderKey = 0;
  let collapsingToLanding = false;
  let playExpandAnimation = false;
  let exitingDeepContent = false;

  function handleMouseMove(e) {
    mouseX = e.clientX; mouseY = e.clientY;
    applyMagnetic(renukaRef); applyMagnetic(joagRef);
    applyMagnetic(curiousRef); applyMagnetic(optimisticRef);
    applyMagnetic(observantRef); applyMagnetic(experimentalRef);
  }
  function applyMagnetic(el) {
    if (!el) return;
    const r = el.getBoundingClientRect();
    const dx = mouseX - (r.left + r.width / 2);
    const dy = mouseY - (r.top + r.height / 2);
    const dist = Math.sqrt(dx*dx + dy*dy);
    if (dist < 180) {
      const s = 1 - dist / 180;
      el.style.transform = `translate(${dx/dist*s*10}px,${dy/dist*s*10}px)`;
    } else { el.style.transform = 'translate(0,0)'; }
  }

  const sections = [
    { label: 'Who I am',    color: '#E35D5B', hasDeepSection: true  },
    { label: 'What I make', color: '#E35D5B', hasDeepSection: false },
    { label: 'How I think', color: '#E35D5B', hasDeepSection: false },
  ];

  let state         = 'landing';
  let expandedIndex = null;
  let expandOrigin  = null;
  let hoveredIndex  = null;
  let showSkipToast = false;

  // Work page: separate mount vs visible so we can crossfade
  let workMounted  = false;   // keeps <Work> in DOM during exit fade
  let workVisible  = false;   // drives opacity
  let workExiting  = false;   // true while fading out

  let deepBgVisible = false;
  let circleGoingToCorner = false;
  let circleComingFromCorner = false;
  let whoIAmComponent;

  let scrollLocked = false;
  function lockScroll(ms = 950) {
    scrollLocked = true;
    setTimeout(() => scrollLocked = false, ms);
  }

  onMount(() => {
    setTimeout(() => { showSkipToast = true; }, 2000);
  });

  function handleWheel(e) {
    if (scrollLocked) return;
    if (state === 'deep') return;

    // Work page: scroll up to go back
    if (workMounted) {
      const delta = Math.abs(e.deltaX) > Math.abs(e.deltaY) ? e.deltaX : e.deltaY;
      if (delta < -25) closeWork();
      return;
    }

    if      (state === 'landing'  && e.deltaY > 30)  goToCircles();
    else if (state === 'circles'  && e.deltaY < -30) goToLanding();
    else if (state === 'expanded' && e.deltaY > 30)  tryGoDeep();
    else if (state === 'expanded' && e.deltaY < -30) goToCircles();
  }

  function goToCircles() {
    if (scrollLocked) return;
    state = 'circles';
    lockScroll(1000);
  }

  function goToLanding() {
    collapsingToLanding = true;
    lockScroll(1000);
    setTimeout(() => {
      state = 'landing';
      collapsingToLanding = false;
    }, 1000);
  }

  function tryGoDeep() {
    if (expandedIndex !== 0) return;
    enterDeep();
  }

  function enterDeep() {
    deepBgVisible = true;
    lockScroll(900);
    setTimeout(() => {
      state = 'deep';
      if (whoIAmComponent) whoIAmComponent.reset();
    }, 120);
  }

  function exitDeep() {
    exitingDeepContent = true;
    lockScroll(1200);
    setTimeout(() => {
      returningFromDeep = true;
      keepCircleVisible = true;
      circleComingFromCorner = true;
    }, 350);
    setTimeout(() => {
      circleComingFromCorner = false;
      deepBgVisible = false;
      state = 'expanded';
      expandedIndex = 0;
      expandOrigin = 0;
    }, 1000);
    setTimeout(() => {
      returningFromDeep = false;
      keepCircleVisible = false;
      exitingDeepContent = false;
    }, 1250);
  }

  function handleCircleClick(i) {
    if (state !== 'circles') return;
    playExpandAnimation = true;
    expandedRenderKey++;
    expandOrigin = i;
    expandedIndex = i;
    state = 'expanded';
    setTimeout(() => { playExpandAnimation = false; }, 800);
  }

  function handleCornerPillClick(i) {
    if (state === 'deep') {
      circleComingFromCorner = true;
      lockScroll(800);
      setTimeout(() => {
        circleComingFromCorner = false;
        deepBgVisible = false;
        playExpandAnimation = true;
        expandedRenderKey++;
        expandOrigin = i;
        expandedIndex = i;
        state = 'expanded';
        setTimeout(() => { playExpandAnimation = false; }, 800);
      }, 700);
      return;
    }
    playExpandAnimation = true;
    expandedRenderKey++;
    expandOrigin = i;
    expandedIndex = i;
    setTimeout(() => { playExpandAnimation = false; }, 800);
  }

  function dismissToast()    { showSkipToast = false; }

  // Open work page: mount → next tick → fade in
  function openWork() {
    showSkipToast = false;
    workMounted = true;
    workExiting = false;
    // Allow DOM to mount before transitioning opacity
    requestAnimationFrame(() => {
      requestAnimationFrame(() => { workVisible = true; });
    });
  }

  // Close work page: fade out → unmount
  function closeWork() {
    workExiting = true;
    workVisible = false;
    lockScroll(700);
    setTimeout(() => {
      workMounted = false;
      workExiting = false;
    }, 650);
  }

  function handlePillAreaClick() { if (state === 'landing') goToCircles(); }

  $: showBigCircle = (state === 'expanded' || circleGoingToCorner) && expandedIndex !== null;
  $: showDeepCornerDot = state === 'deep' || circleComingFromCorner;

  function originClass(o) {
    if (o === 0) return 'from-left';
    if (o === 2) return 'from-right';
    return 'from-center';
  }
</script>

<svelte:window on:mousemove={handleMouseMove} on:wheel|passive={handleWheel} />

<div class="root">

  <!-- ════════ WORK PAGE ════════ -->
  {#if workMounted}
    <div class="work-layer" class:work-layer-visible={workVisible}>
      <Work onBack={closeWork} />
    </div>
  {/if}

  <!-- ════════ LANDING ════════ -->
  <div class="landing-page" class:hidden={state !== 'landing' && !collapsingToLanding}>
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <div class="nav-pills animate-pills" on:click={handlePillAreaClick} style="cursor:pointer">
      <span class="nav-pill"></span><span class="nav-pill"></span><span class="nav-pill"></span>
    </div>
    <h1 class="name renuka animate-left" bind:this={renukaRef} style="transition:transform 0.15s ease-out">Renuka</h1>
    <div class="photo-wrapper animate-photo"><img src={renukaPhoto} alt="Renuka Joag" class="photo" /></div>
    <h1 class="name joag animate-right" bind:this={joagRef} style="transition:transform 0.15s ease-out">Joag</h1>
    <div class="descriptors curious-descriptor"><span class="descriptor animate-desc-left" bind:this={curiousRef} style="transition:transform 0.15s ease-out;animation-delay:1.1s">Curious</span></div>
    <div class="descriptors optimistic-descriptor"><span class="descriptor animate-desc-left" bind:this={optimisticRef} style="transition:transform 0.15s ease-out;animation-delay:1.25s">Optimistic</span></div>
    <div class="descriptors observant-descriptor"><span class="descriptor animate-desc-right" bind:this={observantRef} style="transition:transform 0.15s ease-out;animation-delay:1.1s">Observant</span></div>
    <div class="descriptors experimental-descriptor"><span class="descriptor animate-desc-right" bind:this={experimentalRef} style="transition:transform 0.15s ease-out;animation-delay:1.25s">Experimental</span></div>
    <div class="bottom-bar animate-bar"><span class="bar-text">Portfolio 2026</span></div>
      <div class="toast-card animate-toast">
        <button class="toast-close" on:click={dismissToast} aria-label="Dismiss">✕</button>
        <p class="toast-heading">Short on time?<br>Skip to my work!</p>
        <button class="toast-cta" on:click={openWork}><span>😉</span><span>👌</span></button>
      </div>
  </div>

  <!-- ════════ CIRCLE SELECTION ════════ -->
  <div
    class="selection-viewport"
    class:selection-viewport-active={state !== 'landing' || collapsingToLanding}
    class:selection-viewport-collapsing={collapsingToLanding}
  >
    <CircleSelectionPage
      {state}
      {expandedIndex}
      {expandOrigin}
      {sections}
      {hoveredIndex}
      {circleGoingToCorner}
      {circleComingFromCorner}
      {handleCircleClick}
      {handleCornerPillClick}
      {originClass}
      returningFromDeep={returningFromDeep}
      playExpandAnimation={playExpandAnimation}
    />
    <WhoIAm
      bind:this={whoIAmComponent}
      visible={state === 'deep'}
      bgVisible={deepBgVisible}
      onBack={exitDeep}
      keepCircleVisible={keepCircleVisible}
      exitingContent={exitingDeepContent}
    />
  </div>
</div>

<style>
:global(*, *::before, *::after) { box-sizing: border-box; margin: 0; padding: 0 }
:global(body) { background: #f2eded; overflow: hidden; width: 100vw; height: 100vh }

.root {
  width: 100vw; height: 100vh;
  position: relative; overflow: hidden;
  background: #f2eded;
}

/* ── Work page layer ── */
.work-layer {
  position: absolute;
  inset: 0;
  z-index: 30;
  opacity: 0;
  transition: opacity 0.6s cubic-bezier(0.4, 0, 0.2, 1);
  pointer-events: none;
}
.work-layer.work-layer-visible {
  opacity: 1;
  pointer-events: all;
}

/* ── Landing ── */
.landing-page { position: absolute; inset: 0; z-index: 4; will-change: transform; }
.landing-page.hidden { visibility: hidden; pointer-events: none; }

.nav-pills { position: absolute; top: 4.5vh; left: 4vw; display: flex; gap: 0.7vw; z-index: 10; }
.nav-pill { display: inline-block; width: clamp(18px,2.6vw,38px); height: clamp(18px,2.6vw,38px); border-radius: 50%; background: #E35D5B; }

.name { position: absolute; font-family: 'Trispace',monospace; font-stretch: 112.5%; font-weight: 400; font-size: clamp(80px,15vw,220px); color: #E35D5B; line-height: 1; letter-spacing: -0.01em; user-select: none; will-change: transform; }
.renuka { top: 28%; left: 6vw; z-index: 1; }
.joag   { bottom: 32%; right: 8vw; z-index: 3; }

.photo-wrapper { position: absolute; left: 50%; top: 50%; transform: translate(-50%,-54%); z-index: 2; width: clamp(260px,35vw,520px); }
.photo { width: 100%; height: auto; display: block; object-fit: contain; }

.descriptors { position: absolute; display: flex; flex-direction: column; gap: 0.5vh; z-index: 4; }
.optimistic-descriptor  { left: 30vw; bottom: 28%; }
.curious-descriptor     { left: 36vw; bottom: 48%; }
.observant-descriptor   { right: 32vw; bottom: 28%; text-align: right; align-items: flex-end; }
.experimental-descriptor{ right: 43vw; bottom: 20%; text-align: right; align-items: flex-end; }

.descriptor { font-family: 'Trispace',monospace; font-stretch: 112.5%; font-weight: 300; font-size: clamp(11px,1vw,17px); color: #3A2F2B; letter-spacing: 0.1em; display: inline-block; will-change: transform; opacity: 0; }

.bottom-bar { position: absolute; bottom: 3.5vh; left: 3vw; right: 3vw; height: clamp(36px,5.5vh,58px); background: #7d9e8a; border-radius: 999px; display: flex; align-items: center; justify-content: flex-end; padding-right: 3vw; z-index: 5; }
.bar-text { font-family: 'Trispace',monospace; font-stretch: 112.5%; font-weight: 300; font-size: clamp(10px,1vw,16px); color: #f2eded; letter-spacing: 0.18em; }

.toast-card { position: absolute; top: 4vh; right: 4vw; z-index: 20; width: clamp(220px,22vw,340px); background: #8FAF9A; border-radius: clamp(16px,2vw,28px); padding: clamp(18px,2.2vh,32px) clamp(18px,1.8vw,28px) clamp(14px,1.8vh,24px); display: flex; flex-direction: column; gap: clamp(12px,1.6vh,22px); box-shadow: 0 4px 32px rgba(58,47,43,0.13); }
.toast-close { position: absolute; top: clamp(10px,1.2vh,18px); right: clamp(12px,1.2vw,20px); background: none; border: none; cursor: pointer; font-family: 'Trispace',monospace; font-size: clamp(12px,1.1vw,18px); color: #f2eded; opacity: 0.85; line-height: 1; padding: 2px 4px; transition: opacity 0.15s ease; }
.toast-close:hover { opacity: 1; }
.toast-heading { font-family: 'Trispace',monospace; font-stretch: 112.5%; font-weight: 400; font-size: clamp(13px,1.3vw,20px); color: #f2eded; letter-spacing: 0.04em; line-height: 1.55; padding-right: 1.5em; }
.toast-cta { align-self: flex-start; background: #f2eded; border: none; border-radius: 999px; padding: clamp(6px,0.7vh,10px) clamp(14px,1.2vw,20px); cursor: pointer; display: flex; align-items: center; gap: 0.3em; font-size: clamp(16px,1.6vw,24px); transition: transform 0.2s cubic-bezier(0.34,1.56,0.64,1), box-shadow 0.2s ease; box-shadow: 0 2px 8px rgba(58,47,43,0.08); }
.toast-cta:hover { transform: scale(1.07); box-shadow: 0 4px 16px rgba(58,47,43,0.14); }

/* ── Selection viewport ── */
.selection-viewport { position: absolute; inset: 0; z-index: 8; overflow: hidden; pointer-events: none; transform-origin: 2.45vw 0.7vh; transform: scale(0.08); transition: transform 1s cubic-bezier(0.16,1,0.3,1); will-change: transform; }
.selection-viewport.selection-viewport-active { transform: scale(1); pointer-events: all; }
.selection-viewport.selection-viewport-collapsing { transform: scale(0.12); }

/* ── Shared keyframes ── */
.circles-page { position: absolute; inset: 0; z-index: 2; background: #f2eded; pointer-events: none; display: flex; align-items: center; justify-content: center; }
.circles-page.circles-page-visible { pointer-events: all; }
.corner-pill-slots { position: absolute; top: 4.5vh; left: 4vw; display: flex; gap: 0.7vw; z-index: 12; align-items: center; }
.corner-pill-slot { width: clamp(18px,2.6vw,38px); height: clamp(18px,2.6vw,38px); display: flex; align-items: center; justify-content: center; cursor: pointer; }
.slot-empty { pointer-events: none; }
.corner-circle { width: 100%; height: 100%; border-radius: 50%; background: #E35D5B; transition: transform 0.15s ease; }
.corner-pill-slot:not(.slot-empty):hover .corner-circle { transform: scale(1.2); }
.corner-shrink-0 { animation: shrink0 0.65s cubic-bezier(0.4,0,0.2,1) forwards; }
.corner-shrink-1 { animation: shrink1 0.65s cubic-bezier(0.4,0,0.2,1) forwards; }
.corner-shrink-2 { animation: shrink2 0.65s cubic-bezier(0.4,0,0.2,1) forwards; }
.corner-grow-0   { animation: shrink0 0.65s cubic-bezier(0.4,0,0.2,1) reverse forwards; }
@keyframes shrink0 { from { transform: translate(calc(50vw - 4vw - 1.3vw),calc(50vh - 4.5vh - 1.3vw)) scale(18); } to { transform: translate(0,0) scale(1); } }
@keyframes shrink1 { from { transform: translate(calc(50vw - 4vw - 3.3vw - 1.3vw),calc(50vh - 4.5vh - 1.3vw)) scale(18); } to { transform: translate(0,0) scale(1); } }
@keyframes shrink2 { from { transform: translate(calc(50vw - 4vw - 6.6vw - 1.3vw),calc(50vh - 4.5vh - 1.3vw)) scale(18); } to { transform: translate(0,0) scale(1); } }
.circles-row { display: flex; align-items: center; justify-content: center; gap: 4vw; }
.circle-item { display: flex; flex-direction: column; align-items: center; gap: 2.5vh; }
.big-circle { width: clamp(160px,22vw,340px); height: clamp(160px,22vw,340px); border-radius: 50%; border: none; cursor: pointer; transition: background 0.3s ease, transform 0.3s cubic-bezier(0.34,1.56,0.64,1); display: block; }
.big-circle:hover { transform: scale(1.05); }
.circle-label { font-family: 'Trispace',monospace; font-stretch: 112.5%; font-weight: 300; font-size: clamp(11px,1.1vw,18px); color: #3A2F2B; letter-spacing: 0.12em; }
.expanded-scene { position: absolute; inset: 0; z-index: 10; display: flex; align-items: center; justify-content: center; }
.big-expanded-circle { width: clamp(400px,60vmin,720px); height: clamp(400px,60vmin,720px); border-radius: 50%; display: flex; flex-direction: column; align-items: center; justify-content: center; }
.big-expanded-circle.from-left   { animation: expandFromLeft   0.75s cubic-bezier(0.16,1,0.3,1) forwards; }
.big-expanded-circle.from-center { animation: expandFromCenter 0.75s cubic-bezier(0.16,1,0.3,1) forwards; }
.big-expanded-circle.from-right  { animation: expandFromRight  0.75s cubic-bezier(0.16,1,0.3,1) forwards; }
.big-expanded-circle.circle-to-corner { animation: circleToCorner 0.65s cubic-bezier(0.4,0,0.2,1) forwards !important; }
.expanded-items { display: flex; flex-direction: column; align-items: center; gap: 1.4vh; }
.expanded-item { font-family: 'Trispace',monospace; font-stretch: 112.5%; font-weight: 300; font-size: clamp(13px,1.3vw,22px); color: #f2eded; letter-spacing: 0.1em; opacity: 0; animation: itemFadeIn 0.45s ease forwards; }
.scroll-hint { font-family: 'Trispace',monospace; font-stretch: 112.5%; font-weight: 300; font-size: clamp(9px,0.75vw,13px); color: #f2eded; letter-spacing: 0.15em; opacity: 0; margin-top: 2.5vh; animation: itemFadeIn 0.5s ease 1s forwards; }
@keyframes circleToCorner { from { transform: translate(0,0) scale(1); border-radius: 50%; } to { transform: translate(33vw,34vh) scale(1.15); border-radius: 50%; } }
@keyframes pillsDrop { from { opacity: 0; transform: translateY(-60px); } to { opacity: 1; transform: translateY(0); } }
.animate-pills { opacity: 0; animation: pillsDrop 0.7s cubic-bezier(0.34,1.56,0.64,1) 0.1s forwards; }
@keyframes photoIn { from { opacity: 0; transform: translate(-50%,-50%); } to { opacity: 1; transform: translate(-50%,-54%); } }
.animate-photo { opacity: 0; animation: photoIn 0.9s cubic-bezier(0.25,0.46,0.45,0.94) 0.3s forwards; }
@keyframes slideFromLeft { from { opacity: 0; transform: translateX(-120px); } to { opacity: 1; transform: translateX(0); } }
.animate-left { opacity: 0; animation: slideFromLeft 1s cubic-bezier(0.16,1,0.3,1) 0.5s forwards; }
@keyframes slideFromRight { from { opacity: 0; transform: translateX(120px); } to { opacity: 1; transform: translateX(0); } }
.animate-right { opacity: 0; animation: slideFromRight 1s cubic-bezier(0.16,1,0.3,1) 0.65s forwards; }
@keyframes descFromLeft { from { opacity: 0; transform: translateX(-50px); } to { opacity: 1; transform: translateX(0); } }
.animate-desc-left { animation: descFromLeft 0.8s cubic-bezier(0.16,1,0.3,1) both; }
@keyframes descFromRight { from { opacity: 0; transform: translateX(50px); } to { opacity: 1; transform: translateX(0); } }
.animate-desc-right { animation: descFromRight 0.8s cubic-bezier(0.16,1,0.3,1) both; }
@keyframes barUp { from { opacity: 0; transform: translateY(80px); } to { opacity: 1; transform: translateY(0); } }
.animate-bar { opacity: 0; animation: barUp 0.9s cubic-bezier(0.16,1,0.3,1) 0.8s forwards; }
@keyframes expandFromLeft   { from { opacity: 0.4; transform: translate(-28vw,0) scale(0.22); } to { opacity: 1; transform: translate(0,0) scale(1); } }
@keyframes expandFromCenter { from { opacity: 0.4; transform: scale(0.22); } to { opacity: 1; transform: scale(1); } }
@keyframes expandFromRight  { from { opacity: 0.4; transform: translate(28vw,0) scale(0.22); } to { opacity: 1; transform: translate(0,0) scale(1); } }
@keyframes itemFadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
@keyframes toastSlideIn { from { opacity: 0; transform: translateY(-28px) scale(0.96); } to { opacity: 1; transform: translateY(0) scale(1); } }
.animate-toast { animation: toastSlideIn 0.55s cubic-bezier(0.16,1,0.3,1) forwards; }
</style>
