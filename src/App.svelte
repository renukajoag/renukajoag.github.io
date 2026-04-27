<script>
  import { onMount } from 'svelte';

  // ── Mouse magnetic effect ──
  let mouseX = 0;
  let mouseY = 0;
  let renukaRef, joagRef, curiousRef, optimisticRef, observantRef, experimentalRef;

  function handleMouseMove(e) {
    mouseX = e.clientX;
    mouseY = e.clientY;
    applyMagnetic(renukaRef);
    applyMagnetic(joagRef);
    applyMagnetic(curiousRef);
    applyMagnetic(optimisticRef);
    applyMagnetic(observantRef);
    applyMagnetic(experimentalRef);
  }

  function applyMagnetic(el) {
    if (!el) return;
    const rect = el.getBoundingClientRect();
    const cx = rect.left + rect.width / 2;
    const cy = rect.top + rect.height / 2;
    const dx = mouseX - cx;
    const dy = mouseY - cy;
    const dist = Math.sqrt(dx * dx + dy * dy);
    if (dist < 180) {
      const s = (1 - dist / 180);
      el.style.transform = `translate(${(dx / dist) * s * 10}px, ${(dy / dist) * s * 10}px)`;
    } else {
      el.style.transform = 'translate(0px, 0px)';
    }
  }

  // ── Sections data ──
  const sections = [
    {
      label: 'Who I am',
      color: '#E35D5B',
      items: ['Who I am', 'What makes me, me', 'What I care about', 'Outside of design', 'Right now'],
    },
    {
      label: 'What I make',
      color: '#E35D5B',
      items: ['Brand identity', 'Editorial design', 'Illustration', 'Motion', 'Packaging'],
    },
    {
      label: 'How I think',
      color: '#E35D5B',
      items: ['Research', 'Concept development', 'Systems thinking', 'Iteration', 'Collaboration'],
    },
  ];

  // ── State machine ──
  let state = 'landing'; // 'landing' | 'circles' | 'expanded'
  let expandedIndex = null;
  let hoveredIndex = null;

  let scrollLocked = false;
  function lockScroll(ms = 950) {
    scrollLocked = true;
    setTimeout(() => scrollLocked = false, ms);
  }

  function handleWheel(e) {
    if (scrollLocked) return;
    if (state === 'landing' && e.deltaY > 30) {
      state = 'circles';
      lockScroll();
    } else if (state === 'circles' && e.deltaY < -30) {
      state = 'landing';
      lockScroll();
    } else if (state === 'expanded' && e.deltaY < -30) {
      expandedIndex = null;
      state = 'circles';
      lockScroll(600);
    }
  }

  function handleCircleClick(i) {
    if (state !== 'circles') return;
    expandedIndex = i;
    state = 'expanded';
  }

  function handleCornerPillClick(i) {
    expandedIndex = i;
  }

  $: pillsVisible = state === 'circles' || state === 'expanded';
</script>

<svelte:window on:mousemove={handleMouseMove} on:wheel|passive={handleWheel} />

<div class="root">

  <!-- ══════════ LANDING PAGE ══════════ -->
  <div class="landing-page" class:fade-out={state !== 'landing'}>

    <!-- Three nav pills top left -->
    <div class="nav-pills animate-pills">
      <span class="nav-pill"></span>
      <span class="nav-pill"></span>
      <span class="nav-pill"></span>
    </div>

    <!-- RENUKA: behind photo -->
    <h1 class="name renuka animate-left" bind:this={renukaRef} style="transition: transform 0.15s ease-out;">Renuka</h1>

    <!-- Photo -->
    <div class="photo-wrapper animate-photo">
      <img src="./assets/wbst 1.png" alt="Renuka Joag" class="photo" />
    </div>

    <!-- JOAG: in front of photo -->
    <h1 class="name joag animate-right" bind:this={joagRef} style="transition: transform 0.15s ease-out;">Joag</h1>

    <!-- Descriptors — your exact custom classes preserved -->
    <div class="descriptors curious-descriptor">
      <span class="descriptor animate-desc-left" bind:this={curiousRef} style="transition: transform 0.15s ease-out; animation-delay: 1.1s;">Curious</span>
    </div>
    <div class="descriptors optimistic-descriptor">
      <span class="descriptor animate-desc-left" bind:this={optimisticRef} style="transition: transform 0.15s ease-out; animation-delay: 1.25s;">Optimistic</span>
    </div>
    <div class="descriptors observant-descriptor">
      <span class="descriptor animate-desc-right" bind:this={observantRef} style="transition: transform 0.15s ease-out; animation-delay: 1.1s;">Observant</span>
    </div>
    <div class="descriptors experimental-descriptor">
      <span class="descriptor animate-desc-right" bind:this={experimentalRef} style="transition: transform 0.15s ease-out; animation-delay: 1.25s;">Experimental</span>
    </div>

    <!-- Bottom bar -->
    <div class="bottom-bar animate-bar">
      <span class="bar-text">Portfolio 2026</span>
    </div>

  </div>

  <!--
    ══════════ PILL-BOX OVERLAY ══════════
    This is an absolutely-positioned box that starts small, centred on
    the three dots in the top-left, then expands to fill the whole screen.
    Because it shares the same background colour as the page it looks like
    that corner region is growing.  The three big circles live inside it so
    they appear to grow out of that area as it expands.
  -->
  <div class="pillbox" class:pillbox-open={state === 'circles' || state === 'expanded'}>

    <div class="pills-scene" class:visible={pillsVisible}>

      <!-- Corner pills when one is expanded -->
      {#if state === 'expanded'}
        <div class="corner-pills">
          {#each sections as s, i}
            {#if i !== expandedIndex}
              <button
                class="corner-pill"
                style="background: {s.color};"
                on:click={() => handleCornerPillClick(i)}
                aria-label={s.label}
              ></button>
            {/if}
          {/each}
        </div>
      {/if}

      <!-- Three circles row -->
      <div class="circles-row" class:hidden={state === 'expanded'}>
        {#each sections as s, i}
          <div class="circle-item">
            <button
              class="big-circle"
              style="background: {hoveredIndex === i ? '#8FAF9A' : s.color};"
              on:mouseenter={() => hoveredIndex = i}
              on:mouseleave={() => hoveredIndex = null}
              on:click={() => handleCircleClick(i)}
              aria-label={s.label}
            ></button>
            <span class="circle-label">{s.label}</span>
          </div>
        {/each}
      </div>

      <!-- Expanded circle — centered -->
      {#if state === 'expanded' && expandedIndex !== null}
        {#key expandedIndex}
          <div class="expanded-wrap">
            <div class="expanded-circle" style="background: {sections[expandedIndex].color};">
              <div class="expanded-items">
                {#each sections[expandedIndex].items as item, idx}
                  <span class="expanded-item" style="animation-delay: {0.2 + idx * 0.1}s;">{item}</span>
                {/each}
              </div>
            </div>
          </div>
        {/key}
      {/if}

    </div>
  </div>

</div>

<style>
  :global(*, *::before, *::after) { box-sizing: border-box; margin: 0; padding: 0; }
  :global(body) {
    background: #f2eded;
    overflow: hidden;
    width: 100vw;
    height: 100vh;
  }

  .root {
    width: 100vw;
    height: 100vh;
    position: relative;
    overflow: hidden;
    background: #f2eded;
  }

  /* ══════════════════════════════
     LANDING — all positions exactly
     as they were before this change
  ══════════════════════════════ */
  .landing-page {
    position: absolute;
    inset: 0;
    z-index: 2;
    transition: opacity 0.4s ease;
  }
  .landing-page.fade-out {
    opacity: 0;
    pointer-events: none;
  }

  .nav-pills {
    position: absolute;
    top: 4.5vh;
    left: 4vw;
    display: flex;
    gap: 0.7vw;
    z-index: 10;
  }
  .nav-pill {
    display: inline-block;
    width: clamp(18px, 2.6vw, 38px);
    height: clamp(18px, 2.6vw, 38px);
    border-radius: 50%;
    background: #E35D5B;
  }

  .name {
    position: absolute;
    font-family: 'Trispace', monospace;
    font-stretch: 112.5%;
    font-weight: 400;
    font-size: clamp(80px, 15vw, 220px);
    color: #E35D5B;
    line-height: 1;
    letter-spacing: -0.01em;
    user-select: none;
    will-change: transform;
  }
  .renuka { top: 28%; left: 6vw; z-index: 1; }
  .joag   { bottom: 32%; right: 8vw; z-index: 3; }

  .photo-wrapper {
    position: absolute;
    left: 50%; top: 50%;
    transform: translate(-50%, -54%);
    z-index: 2;
    width: clamp(260px, 35vw, 520px);
  }
  .photo { width: 100%; height: auto; display: block; object-fit: contain; }

  /* Descriptor container */
  .descriptors {
    position: absolute;
    display: flex;
    flex-direction: column;
    gap: 0.5vh;
    z-index: 4;
  }

  /* Your exact custom positioning classes — untouched */
  .optimistic-descriptor  { left: 30vw; bottom: 28%; }
  .curious-descriptor     { left: 36vw; bottom: 48%; }
  .observant-descriptor   { right: 32vw; bottom: 28%; text-align: right; align-items: flex-end; }
  .experimental-descriptor { right: 43vw; bottom: 20%; text-align: right; align-items: flex-end; }

  .descriptor {
    font-family: 'Trispace', monospace;
    font-stretch: 112.5%;
    font-weight: 300;
    font-size: clamp(11px, 1vw, 17px);
    color: #3A2F2B;
    letter-spacing: 0.1em;
    display: inline-block;
    will-change: transform;
    opacity: 0;
  }

  .bottom-bar {
    position: absolute;
    bottom: 3.5vh; left: 3vw; right: 3vw;
    height: clamp(36px, 5.5vh, 58px);
    background: #7d9e8a;
    border-radius: 999px;
    display: flex;
    align-items: center;
    justify-content: flex-end;
    padding-right: 3vw;
    z-index: 5;
  }
  .bar-text {
    font-family: 'Trispace', monospace;
    font-stretch: 112.5%;
    font-weight: 300;
    font-size: clamp(10px, 1vw, 16px);
    color: #f2eded;
    letter-spacing: 0.18em;
  }

  /* ══════════════════════════════════════════════
     PILLBOX
     Starts as a small rounded rectangle centred on
     the three dots. Expands to fill the viewport.

     The dots sit at:
       left edge  ≈ 4vw
       top edge   ≈ 4.5vh
       width      ≈ three dots + two gaps
                  ≈ 3 × 2.6vw + 2 × 0.7vw = 9.2vw
       height     ≈ 2.6vw (dot diameter)

     So the box centre is approximately:
       cx = 4vw + 9.2vw/2  = 8.6vw
       cy = 4.5vh + 1.3vw/2 ≈ 5.1vh  (1.3vw ≈ half dot, treat as ~1vh)

     We position the pillbox with top/left at that centre
     and use translate(-50%,-50%) so it truly centres there.
  ══════════════════════════════════════════════ */
  .pillbox {
    position: absolute;
    /* Centre point of the three-dot cluster */
    top: 5.1vh;
    left: 8.6vw;
    transform: translate(-50%, -50%) scaleX(1) scaleY(1);

    /* Collapsed: small box that just wraps the dots */
    width: 10vw;
    height: 4vh;
    border-radius: 999px;

    z-index: 3;
    overflow: hidden;

    /* Smooth expansion on both axes */
    transition:
      width  0.85s cubic-bezier(0.4, 0, 0.2, 1),
      height 0.85s cubic-bezier(0.4, 0, 0.2, 1),
      top    0.85s cubic-bezier(0.4, 0, 0.2, 1),
      left   0.85s cubic-bezier(0.4, 0, 0.2, 1),
      border-radius 0.85s cubic-bezier(0.4, 0, 0.2, 1);
  }

  /* Expanded: fills the full viewport */
  .pillbox.pillbox-open {
    top: 50vh;
    left: 50vw;
    width: 100vw;
    height: 100vh;
    border-radius: 0;
  }

  /* ══════════════════════
     PILLS / CIRCLES SCENE
  ══════════════════════ */
  .pills-scene {
    position: absolute;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    opacity: 0;
    pointer-events: none;
    /* Fade in only after the box has mostly expanded */
    transition: opacity 0.35s ease 0.6s;
  }
  .pills-scene.visible {
    opacity: 1;
    pointer-events: all;
  }

  .corner-pills {
    position: absolute;
    top: 4.5vh;
    left: 4vw;
    display: flex;
    gap: 0.7vw;
    z-index: 10;
    animation: fadeIn 0.4s ease forwards;
  }
  .corner-pill {
    width: clamp(18px, 2.4vw, 36px);
    height: clamp(18px, 2.4vw, 36px);
    border-radius: 50%;
    border: none;
    cursor: pointer;
    transition: transform 0.2s ease;
  }
  .corner-pill:hover { transform: scale(1.18); }

  .circles-row {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 4vw;
    transition: opacity 0.35s ease;
  }
  .circles-row.hidden { opacity: 0; pointer-events: none; }

  .circle-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2.5vh;
    animation: circleEntrance 0.7s cubic-bezier(0.16, 1, 0.3, 1) both;
  }
  .circle-item:nth-child(1) { animation-delay: 0.55s; }
  .circle-item:nth-child(2) { animation-delay: 0.65s; }
  .circle-item:nth-child(3) { animation-delay: 0.75s; }

  .big-circle {
    width: clamp(160px, 22vw, 340px);
    height: clamp(160px, 22vw, 340px);
    border-radius: 50%;
    border: none;
    cursor: pointer;
    transition: background 0.3s ease, transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
    display: block;
  }
  .big-circle:hover { transform: scale(1.05); }

  .circle-label {
    font-family: 'Trispace', monospace;
    font-stretch: 112.5%;
    font-weight: 300;
    font-size: clamp(11px, 1.1vw, 18px);
    color: #3A2F2B;
    letter-spacing: 0.12em;
  }

  .expanded-wrap {
    position: absolute;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    animation: expandWrap 0.75s cubic-bezier(0.16, 1, 0.3, 1) forwards;
  }
  .expanded-circle {
    width: clamp(400px, 60vmin, 720px);
    height: clamp(400px, 60vmin, 720px);
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .expanded-items {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1.4vh;
  }
  .expanded-item {
    font-family: 'Trispace', monospace;
    font-stretch: 112.5%;
    font-weight: 300;
    font-size: clamp(13px, 1.3vw, 22px);
    color: #f2eded;
    letter-spacing: 0.1em;
    opacity: 0;
    animation: itemFadeIn 0.45s ease forwards;
  }

  /* ══════════ ANIMATIONS ══════════ */
  @keyframes pillsDrop {
    from { opacity: 0; transform: translateY(-60px); }
    to   { opacity: 1; transform: translateY(0); }
  }
  .animate-pills {
    opacity: 0;
    animation: pillsDrop 0.7s cubic-bezier(0.34, 1.56, 0.64, 1) 0.1s forwards;
  }

  @keyframes photoIn {
    from { opacity: 0; transform: translate(-50%, -50%); }
    to   { opacity: 1; transform: translate(-50%, -54%); }
  }
  .animate-photo {
    opacity: 0;
    animation: photoIn 0.9s cubic-bezier(0.25, 0.46, 0.45, 0.94) 0.3s forwards;
  }

  @keyframes slideFromLeft {
    from { opacity: 0; transform: translateX(-120px); }
    to   { opacity: 1; transform: translateX(0); }
  }
  .animate-left {
    opacity: 0;
    animation: slideFromLeft 1s cubic-bezier(0.16, 1, 0.3, 1) 0.5s forwards;
  }

  @keyframes slideFromRight {
    from { opacity: 0; transform: translateX(120px); }
    to   { opacity: 1; transform: translateX(0); }
  }
  .animate-right {
    opacity: 0;
    animation: slideFromRight 1s cubic-bezier(0.16, 1, 0.3, 1) 0.65s forwards;
  }

  @keyframes descFromLeft {
    from { opacity: 0; transform: translateX(-50px); }
    to   { opacity: 1; transform: translateX(0); }
  }
  .animate-desc-left { animation: descFromLeft 0.8s cubic-bezier(0.16, 1, 0.3, 1) both; }

  @keyframes descFromRight {
    from { opacity: 0; transform: translateX(50px); }
    to   { opacity: 1; transform: translateX(0); }
  }
  .animate-desc-right { animation: descFromRight 0.8s cubic-bezier(0.16, 1, 0.3, 1) both; }

  @keyframes barUp {
    from { opacity: 0; transform: translateY(80px); }
    to   { opacity: 1; transform: translateY(0); }
  }
  .animate-bar {
    opacity: 0;
    animation: barUp 0.9s cubic-bezier(0.16, 1, 0.3, 1) 0.8s forwards;
  }

  @keyframes circleEntrance {
    from { opacity: 0; transform: scale(0.7) translateY(20px); }
    to   { opacity: 1; transform: scale(1) translateY(0); }
  }

  @keyframes expandWrap {
    from { transform: scale(0.28); opacity: 0.4; }
    to   { transform: scale(1); opacity: 1; }
  }

  @keyframes itemFadeIn {
    from { opacity: 0; transform: translateY(10px); }
    to   { opacity: 1; transform: translateY(0); }
  }

  @keyframes fadeIn {
    from { opacity: 0; }
    to   { opacity: 1; }
  }
</style>
