<script>
  import { onMount } from 'svelte';

  // Mouse tracking for magnetic text effect
  let mouseX = 0;
  let mouseY = 0;

  // Refs for magnetic text elements
  let renukaRef;
  let joagRef;
  let curiousRef;
  let optimisticRef;
  let observantRef;
  let experimentalRef;

  function handleMouseMove(e) {
    mouseX = e.clientX;
    mouseY = e.clientY;
    applyMagneticEffect(renukaRef);
    applyMagneticEffect(joagRef);
    applyMagneticEffect(curiousRef);
    applyMagneticEffect(optimisticRef);
    applyMagneticEffect(observantRef);
    applyMagneticEffect(experimentalRef);
  }

  function applyMagneticEffect(el) {
    if (!el) return;
    const rect = el.getBoundingClientRect();
    const elCenterX = rect.left + rect.width / 2;
    const elCenterY = rect.top + rect.height / 2;
    const dx = mouseX - elCenterX;
    const dy = mouseY - elCenterY;
    const dist = Math.sqrt(dx * dx + dy * dy);
    const maxDist = 180;
    const maxShift = 10;

    if (dist < maxDist) {
      const strength = (1 - dist / maxDist);
      const shiftX = (dx / dist) * strength * maxShift;
      const shiftY = (dy / dist) * strength * maxShift;
      el.style.transform = `translate(${shiftX}px, ${shiftY}px)`;
    } else {
      el.style.transform = `translate(0px, 0px)`;
    }
  }
</script>

<svelte:window on:mousemove={handleMouseMove} />

<main class="landing">

  <!-- Three pills top left -->
  <div class="pills-container animate-pills">
    <span class="pill"></span>
    <span class="pill"></span>
    <span class="pill"></span>
  </div>

  <!-- RENUKA: behind photo, slides in from left -->
  <h1
    class="name renuka animate-left"
    bind:this={renukaRef}
    style="transition: transform 0.15s ease-out;"
  >
    Renuka
  </h1>

  <!-- Photo layer: sits between Renuka (behind) and Joag (front) -->
  <div class="photo-wrapper animate-photo">
    <img src="./assets/wbst 1.png" alt="Renuka Joag" class="photo" />
  </div>

  <!-- JOAG: in front of photo, slides in from right -->
  <h1
    class="name joag animate-right"
    bind:this={joagRef}
    style="transition: transform 0.15s ease-out;"
  >
    Joag
  </h1>

  <!-- Descriptor words, left side -->
  <div class="descriptors left-descriptors">
    <span class="descriptor animate-desc-left" bind:this={curiousRef} style="transition: transform 0.15s ease-out; animation-delay: 1.1s;">Curious</span>
    <span class="descriptor animate-desc-left" bind:this={optimisticRef} style="transition: transform 0.15s ease-out; animation-delay: 1.25s;">Optimistic</span>
  </div>

  <!-- Descriptor words, right side -->
  <div class="descriptors right-descriptors">
    <span class="descriptor animate-desc-right" bind:this={observantRef} style="transition: transform 0.15s ease-out; animation-delay: 1.1s;">Observant</span>
    <span class="descriptor animate-desc-right" bind:this={experimentalRef} style="transition: transform 0.15s ease-out; animation-delay: 1.25s;">Experimental</span>
  </div>

  <!-- Bottom bar -->
  <div class="bottom-bar animate-bar">
    <span class="bar-text">Portfolio 2026</span>
  </div>

</main>

<style>
  :global(*, *::before, *::after) {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  :global(body) {
    background: #f2eded;
    overflow: hidden;
    width: 100vw;
    height: 100vh;
  }

  .landing {
    position: relative;
    width: 100vw;
    height: 100vh;
    background: #f2eded;
    overflow: hidden;
    font-family: 'Trispace', monospace;
    font-stretch: 112.5%;
  }

  /* ── PILLS ── */
  .pills-container {
    position: absolute;
    top: 4.5vh;
    left: 4vw;
    display: flex;
    gap: 0.7vw;
    z-index: 10;
  }

  .pill {
    display: inline-block;
    width: clamp(18px, 2.6vw, 38px);
    height: clamp(18px, 2.6vw, 38px);
    border-radius: 50%;
    background: #E35D5B;
  }

  /* ── NAME TEXT ── */
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

  .renuka {
    top: 28%;
    left: 3vw;
    z-index: 1; /* behind photo */
  }

  .joag {
    bottom: 16%;
    right: 2vw;
    z-index: 3; /* in front of photo */
  }

  /* ── PHOTO ── */
  .photo-wrapper {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -54%);
    z-index: 2;
    width: clamp(260px, 35vw, 520px);
  }

  .photo {
    width: 100%;
    height: auto;
    display: block;
    object-fit: contain;
  }

  /* ── DESCRIPTORS ── */
  .descriptors {
    position: absolute;
    display: flex;
    flex-direction: column;
    gap: 0.4vh;
    z-index: 4;
  }

  .left-descriptors {
    left: 4vw;
    bottom: 18%;
  }

  .right-descriptors {
    right: 4vw;
    top: 12%;
    text-align: right;
    align-items: flex-end;
  }

  .descriptor {
    font-family: 'Trispace', monospace;
    font-stretch: 112.5%;
    font-weight: 300;
    font-size: clamp(11px, 1.1vw, 18px);
    color: #E35D5B;
    letter-spacing: 0.12em;
    text-transform: uppercase;
    display: inline-block;
    will-change: transform;
    opacity: 0;
  }

  /* ── BOTTOM BAR ── */
  .bottom-bar {
    position: absolute;
    bottom: 3.5vh;
    left: 3vw;
    right: 3vw;
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

  /* ════════════════════════════
     ANIMATIONS
  ════════════════════════════ */

  /* Pills: drop in from top */
  @keyframes pillsDrop {
    from {
      opacity: 0;
      transform: translateY(-60px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .animate-pills {
    opacity: 0;
    animation: pillsDrop 0.7s cubic-bezier(0.34, 1.56, 0.64, 1) 0.1s forwards;
  }

  /* Photo: fade + subtle scale in */
  @keyframes photoIn {
    from {
      opacity: 0;
      transform: translate(-50%, -50%);
    }
    to {
      opacity: 1;
      transform: translate(-50%, -54%);
    }
  }

  .animate-photo {
    opacity: 0;
    animation: photoIn 0.9s cubic-bezier(0.25, 0.46, 0.45, 0.94) 0.3s forwards;
  }

  /* Renuka: slide in from left */
  @keyframes slideFromLeft {
    from {
      opacity: 0;
      transform: translateX(-120px);
    }
    to {
      opacity: 1;
      transform: translateX(0);
    }
  }

  .animate-left {
    opacity: 0;
    animation: slideFromLeft 1s cubic-bezier(0.16, 1, 0.3, 1) 0.5s forwards;
  }

  /* Joag: slide in from right */
  @keyframes slideFromRight {
    from {
      opacity: 0;
      transform: translateX(120px);
    }
    to {
      opacity: 1;
      transform: translateX(0);
    }
  }

  .animate-right {
    opacity: 0;
    animation: slideFromRight 1s cubic-bezier(0.16, 1, 0.3, 1) 0.65s forwards;
  }

  /* Descriptors left: slide from left */
  @keyframes descFromLeft {
    from {
      opacity: 0;
      transform: translateX(-50px);
    }
    to {
      opacity: 1;
      transform: translateX(0);
    }
  }

  .animate-desc-left {
    animation: descFromLeft 0.8s cubic-bezier(0.16, 1, 0.3, 1) both;
  }

  /* Descriptors right: slide from right */
  @keyframes descFromRight {
    from {
      opacity: 0;
      transform: translateX(50px);
    }
    to {
      opacity: 1;
      transform: translateX(0);
    }
  }

  .animate-desc-right {
    animation: descFromRight 0.8s cubic-bezier(0.16, 1, 0.3, 1) both;
  }

  /* Bottom bar: slide up from bottom */
  @keyframes barUp {
    from {
      opacity: 0;
      transform: translateY(80px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .animate-bar {
    opacity: 0;
    animation: barUp 0.9s cubic-bezier(0.16, 1, 0.3, 1) 0.8s forwards;
  }
</style>
