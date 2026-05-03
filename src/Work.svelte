<script>
  import Background from './Background.svelte';
  import work1 from '../assets/work1.png';
  import work2 from '../assets/work2.png';
  import work3 from '../assets/work3.png';
  import work4 from '../assets/work4.png';
  import work5 from '../assets/work5.png';
  import work6 from '../assets/work6.png';

  export let onBack = () => {};

  const projects = [
    {
      image: work1,
      line1: 'Book in Progress–',
      line2: 'Layouting, Publication design',
      behanceUrl: 'https://www.behance.net/renukajoag1/projects',
    },
    {
      image: work2,
      line1: 'Ono –',
      line2: 'Branding, Packaging',
      behanceUrl: 'https://www.behance.net/renukajoag1/projects',
    },
    {
      image: work3,
      line1: 'Poster Playground –',
      line2: 'Experimenting, Challenging myself',
      behanceUrl: 'https://www.behance.net/gallery/237097785/Poster-Playground',
    },
    {
      image: work4,
      line1: 'Crochet Lookbook -',
      line2: 'Layout, Passion project',
      behanceUrl: 'https://www.behance.net/gallery/236913297/My-Crochet-Lookbook',
    },
    {
      image: work5,
      line1: 'Naviin –',
      line2: 'Branding, Packaging',
      behanceUrl: 'https://www.behance.net/gallery/237093557/Naviin-Branding',
    },
    {
      image: work6,
      line1: 'Mochi : Mental Health Journal –',
      line2: 'Research, Book design',
      behanceUrl: 'https://www.behance.net/gallery/237100855/Mochi-Mental-Health-Journal',
    },
  ];

  const N = projects.length;
  let current = 0;
  let direction = 1;
  let animKey = 0;

  let scrollLocked = false;
  function lockScroll(ms = 650) {
    scrollLocked = true;
    setTimeout(() => scrollLocked = false, ms);
  }

  function prev() {
    if (scrollLocked) return;
    direction = -1;
    current = (current - 1 + N) % N;
    animKey++;
    lockScroll();
  }

  function next() {
    if (scrollLocked) return;
    direction = 1;
    current = (current + 1) % N;
    animKey++;
    lockScroll();
  }

  function handleWheel(e) {
    const delta = Math.abs(e.deltaX) > Math.abs(e.deltaY) ? e.deltaX : e.deltaY;
    if (delta > 25)       next();
    else if (delta < -25) prev();
  }

  function handleKey(e) {
    if (e.key === 'ArrowRight' || e.key === 'ArrowDown') next();
    if (e.key === 'ArrowLeft'  || e.key === 'ArrowUp')   prev();
  }

  $: leftIndex     = (current - 1 + N) % N;
  $: rightIndex    = (current + 1) % N;
  $: centerProject = projects[current];
  $: leftProject   = projects[leftIndex];
  $: rightProject  = projects[rightIndex];
</script>

<svelte:window on:keydown={handleKey} />

<!-- svelte-ignore a11y-no-static-element-interactions -->
<div class="work-root" on:wheel|preventDefault={handleWheel}>

  <Background />

  <!--
    Layout (left→right):
    [8.5% gap] [left-img] [arrow-left] [center-img+title] [arrow-right] [right-img] [8.5% gap]

    We use a CSS grid with named columns so the arrows sit
    visually between the images rather than at the screen edges.
  -->
  <div class="stage">

    <!-- Left image — starts at 8.5% from left edge -->
    <div class="slot slot-left">
      <button class="side-btn" on:click={prev} aria-label="Previous project">
        <img src={leftProject.image} alt={leftProject.line1} class="card-img side-img" draggable="false" />
      </button>
    </div>

    <!-- Left arrow — between left image and center -->
    <div class="slot slot-arrow-left">
      <button class="arrow" on:click={prev} aria-label="Previous">‹</button>
    </div>

    <!-- Center image + title -->
    <div class="slot slot-center">
      {#key animKey}
        <div class="center-card {direction === 1 ? 'enter-from-right' : 'enter-from-left'}">
          <a
            href={centerProject.behanceUrl}
            target="_blank"
            rel="noopener noreferrer"
            class="card-link"
            aria-label={centerProject.line1}
          >
            <img
              src={centerProject.image}
              alt={centerProject.line1}
              class="card-img center-img"
              draggable="false"
            />
          </a>
          <div class="card-title">
            <span class="title-line">{centerProject.line1}</span>
            <span class="title-line">{centerProject.line2}</span>
          </div>
        </div>
      {/key}
    </div>

    <!-- Right arrow — between center and right image -->
    <div class="slot slot-arrow-right">
      <button class="arrow" on:click={next} aria-label="Next">›</button>
    </div>

    <!-- Right image — ends at 8.5% from right edge -->
    <div class="slot slot-right">
      <button class="side-btn" on:click={next} aria-label="Next project">
        <img src={rightProject.image} alt={rightProject.line1} class="card-img side-img" draggable="false" />
      </button>
    </div>

  </div>

</div>

<style>
  .work-root {
    position: absolute;
    inset: 0;
    z-index: 20;
    overflow: hidden;
    background: #f2eded;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  /*
    Grid columns:
    8.5vw  |  side-img  |  arrow  |  center  |  arrow  |  side-img  |  8.5vw
    The side images share whatever space remains after the center and margins.
    Center is given a fixed preferred width; side images take equal shares of the rest.
  */
  .stage {
    position: relative;
    z-index: 2;
    width: 100%;
    height: 100%;
    display: grid;
    grid-template-columns:
      8.5vw          /* left margin  */
      1fr            /* left image   */
      clamp(28px, 3.5vw, 56px)   /* left arrow   */
      clamp(200px, 30vw, 460px)  /* center image */
      clamp(28px, 3.5vw, 56px)   /* right arrow  */
      1fr            /* right image  */
      8.5vw;         /* right margin */
    grid-template-rows: 1fr;
    align-items: center;
  }

  /* Slot helpers — map children to grid columns */
  .slot { display: flex; align-items: center; justify-content: center; height: 100%; }
  .slot-left        { grid-column: 2; }
  .slot-arrow-left  { grid-column: 3; }
  .slot-center      { grid-column: 4; flex-direction: column; }
  .slot-arrow-right { grid-column: 5; }
  .slot-right       { grid-column: 6; }

  /* ── Side images ── */
  .side-btn {
    background: none;
    border: none;
    padding: 0;
    cursor: pointer;
    width: 100%;
    max-width: clamp(90px, 14vw, 220px);
    outline: none;
  }

  .side-img {
    opacity: 0.68;
    transition: opacity 0.2s ease, transform 0.2s ease;
  }
  .side-btn:hover .side-img {
    opacity: 1;
    transform: scale(1.03);
  }

  /* ── Center card ── */
  .center-card {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2.2vh;
    width: 100%;
  }

  /* ── Images shared styles ── */
  .card-img {
    width: 100%;
    height: auto;
    display: block;
    object-fit: cover;
    aspect-ratio: 1 / 1;
    box-shadow: 19px 19px 12px rgba(0, 0, 0, 0.25);
    border-radius: 2px;
    user-select: none;
  }

  .card-link {
    display: block;
    width: 100%;
    cursor: pointer;
    transition: transform 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);
  }
  .card-link:hover { transform: scale(1.025); }
  .card-link:hover .center-img {
    box-shadow: 22px 22px 18px rgba(0, 0, 0, 0.28);
  }

  /* ── Title ── */
  .card-title {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.15em;
    text-align: center;
  }
  .title-line {
    font-family: 'Trispace', monospace;
    font-stretch: 112.5%;
    font-weight: 300;
    font-size: clamp(10px, 0.95vw, 16px);
    color: #3A2F2B;
    letter-spacing: 0.1em;
    line-height: 1.5;
  }

  /* ── Arrows — sit between images ── */
  .arrow {
    background: none;
    border: none;
    cursor: pointer;
    font-family: 'Trispace', monospace;
    font-size: clamp(20px, 2.2vw, 38px);
    color: #3A2F2B;
    opacity: 0.38;
    line-height: 1;
    user-select: none;
    padding: 0;
    transition: opacity 0.2s ease, transform 0.15s ease;
  }
  .arrow:hover {
    opacity: 0.85;
    transform: scale(1.15);
  }

  /* ── Center card entrance animations ── */
  @keyframes enterFromRight {
    from { opacity: 0; transform: translateX(52px); }
    to   { opacity: 1; transform: translateX(0); }
  }
  @keyframes enterFromLeft {
    from { opacity: 0; transform: translateX(-52px); }
    to   { opacity: 1; transform: translateX(0); }
  }
  .center-card.enter-from-right {
    animation: enterFromRight 0.55s cubic-bezier(0.16, 1, 0.3, 1) both;
  }
  .center-card.enter-from-left {
    animation: enterFromLeft 0.55s cubic-bezier(0.16, 1, 0.3, 1) both;
  }
</style>
