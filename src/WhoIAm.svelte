<script>
  export let visible    = false;
  export let bgVisible  = false;
  export let onBack     = () => {};
import renukaWhoIAm from "../assets/renuka_whoiam.png";

  // ── Card data ──
  const cards = [
    {
      title: 'Who I am',
      body: `I'm someone who tries to stay optimistic and I have a lot of different interests, hobbies, and people in my life that shape how I think and what I notice. 

I tend to pick up on small details and hold onto them. I like reading, writing, and crocheting, and I think all of these things influence how I see the world and what I end up creating.`,
      hasPhoto: true,
    },
    {
      title: 'What makes me, me',
      body: `I genuinely like doing things that make someone feel a little better, even if it's something small. That matters to me more than big outcomes. 

I think empathy plays a big role in how I approach both people and design, and I'm naturally drawn to finding meaning in everyday situations.`,
      hasPhoto: false,
    },
    {
      title: 'What I care about',
      body: `I care a lot about designing with intent. For me, it's not just about making something look good, it should have a reason behind it or create some kind of impact. I like the idea that design can be a way to contribute, even in small ways, and that's something I try to keep in mind while working.`,
      hasPhoto: false,
    },
    {
      title: 'Outside of design',
      body: `Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.`,
      hasPhoto: false,
    },
    {
      title: 'Right now',
      body: `Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.`,
      hasPhoto: false,
    },
  ];

  let currentCard = 0;
  // direction: 1 = going forward (text comes from right), -1 = going back (text comes from left)
  let direction = 1;
  // animKey forces re-mount of text elements to re-trigger animations
  let animKey = 0;

  export function reset() {
    currentCard = 0;
    direction = 1;
    animKey++;
  }

  let scrollLocked = false;
  function lockScroll(ms = 700) {
    scrollLocked = true;
    setTimeout(() => scrollLocked = false, ms);
  }

  function goNext() {
    if (!visible || scrollLocked) return;
    if (currentCard < cards.length - 1) {
      direction = 1;
      currentCard++;
      animKey++;
      lockScroll();
    }
  }
  function goPrev() {
    if (!visible || scrollLocked) return;
    if (currentCard > 0) {
      direction = -1;
      currentCard--;
      animKey++;
      lockScroll();
    } else {
      onBack();
    }
  }

  function handleWheel(e) {
    if (!visible || scrollLocked) return;
    // Support both vertical (trackpad swipe up/down) and horizontal (trackpad swipe left/right)
    const delta = Math.abs(e.deltaX) > Math.abs(e.deltaY) ? e.deltaX : e.deltaY;
    if (delta > 25)       goNext();
    else if (delta < -25) goPrev();
  }

  function handleKey(e) {
    if (!visible) return;
    if (e.key === 'ArrowRight' || e.key === 'ArrowDown') goNext();
    if (e.key === 'ArrowLeft'  || e.key === 'ArrowUp')   goPrev();
  }
</script>

<svelte:window on:keydown={handleKey} />

<!-- svelte-ignore a11y-no-static-element-interactions -->
<div
  class="wia-root"
  class:bg-visible={bgVisible || visible}
  class:content-visible={visible}
  on:wheel|preventDefault={handleWheel}
>

  <!-- ── Gradient background ── -->
  <!-- Fades in as soon as bgVisible is true (while circle is still travelling) -->
  <div class="wia-bg"></div>


  <!--
    ── Static decorative layer ──
    The red circle and the photo never move.
    They sit in their fixed positions on the page at all times.
    Only the text animates.
  -->

  <!-- Decorative red circle — bottom right, partially clipped -->
  <div class="deco-circle"
        class:hidden-deco={!visible}
        >
    </div>

  <!-- Photo — only rendered always but visible only when card 0 is active -->
  <img
    src={renukaWhoIAm}
    alt="Renuka"
    class="deco-photo"
    class:photo-visible={currentCard === 0 && visible}
  />

  <!--
    ── Text layer ──
    Title and body re-mount with {#key animKey} so they always
    re-trigger their entrance animation.
    direction=1  → text slides in from right  (going forward)
    direction=-1 → text slides in from left   (going back)
  -->
  <div class="text-layer">
    {#key animKey}
      <h2
        class="card-title"
        class:enter-from-right={direction === 1}
        class:enter-from-left={direction === -1}
      >
        {cards[currentCard].title}
      </h2>

      <p
        class="card-body"
        class:enter-from-right={direction === 1}
        class:enter-from-left={direction === -1}
        style="animation-delay: 0.08s"
      >
        {cards[currentCard].body}
      </p>
    {/key}
  </div>

  <!-- ── Progress dots ── -->
  <div class="progress-dots">
    {#each cards as _, i}
      <button
        class="prog-dot"
        class:active={i === currentCard}
        on:click={() => {
          direction = i > currentCard ? 1 : -1;
          currentCard = i;
          animKey++;
        }}
        aria-label="Card {i+1}"
      ></button>
    {/each}
  </div>

</div>

<style>
  /* z:5 — above circles page (z:2), below travelling circle (z:12) */
  .wia-root {
    position: absolute;
    inset: 0;
    z-index: 5;
    pointer-events: none;
    overflow: hidden;
  }

  /* ── Gradient ──
     Top to bottom: #F6EDEE at 20% stop → #8FAF9A at 100%
  */
  .wia-bg {
    position: absolute;
    inset: 0;
    z-index: 0;
    background: linear-gradient(to bottom, #F6EDEE 20%, #8FAF9A 100%);
    opacity: 0;
    transition: opacity 0.6s cubic-bezier(0.4, 0, 0.2, 1);
  }
  .wia-root.bg-visible .wia-bg { opacity: 1; }

  /* Content fades in after bg is mostly in */
  .text-layer,
  .progress-dots,
  .deco-photo {
    opacity: 0;
    transition: opacity 0.35s ease 0.35s;
  }
  .wia-root.content-visible .wia-dots,
  .wia-root.content-visible .text-layer,
  .wia-root.content-visible .progress-dots,
  .wia-root.content-visible .deco-circle {
    opacity: 1;
    pointer-events: all;
  }
  /* Photo has its own visibility tied to currentCard */
  .deco-photo {
    transition: opacity 0.4s ease;
  }
  .deco-photo.photo-visible { opacity: 1; }


  /* ── Decorative circle ──
     Fixed bottom-right. Size and position set to match the
     reference image at 1660×1033 canvas.
     Circle is roughly 44vw wide, bottom edge at -10vw, right edge at -10vw.
  */
  .deco-circle {
    position: absolute;
    bottom: -10vw;
    right: -14vw;
    width: clamp(300px, 44vw, 680px);
    height: clamp(300px, 44vw, 680px);
    border-radius: 50%;
    background: #E35D5B;
    z-index: 1;
    pointer-events: none;
  }

.hidden-deco {
  opacity: 0;
}
  /* ── Photo ──
     Fixed bottom-right, in front of deco circle.
     Tall portrait image, bottom-aligned.
     right: 6vw places it over the left portion of the circle.
  */
.deco-photo{
  position:absolute;
  bottom:-6vh;
  right:5vw;

  height:clamp(821px,75vh,1095px);
  width:auto;

  object-fit:contain;
  object-position:bottom;

  z-index:2;

  pointer-events:none;

  opacity:0;

  transform:translateY(10vh);

  transition:
    opacity 0.45s ease,
    transform 0.75s cubic-bezier(0.16,1,0.3,1);
}

.deco-photo.photo-visible{
  opacity:1;
  transform:translateY(0);
}

  /* ── Text layer ──
     Sits above everything except the dots.
  */
  .text-layer {
    position: absolute;
    inset: 0;
    z-index: 3;
    pointer-events: none;
  }

  /* ── Title ──
     Large, upper portion. Left-aligned.
     ~clamp 60px–120px. Top: 14vh.
     Wide letter-spacing like the reference "W h o   I   a m".
  */
  .card-title {
    position: absolute;
    top: 14vh;
    left: 5vw;
    /* leave room so it doesn't go behind the photo area */
    max-width: 95vw;
    font-family: 'Trispace', monospace;
    font-stretch: 112.5%;
    font-weight: 400;
    font-size: clamp(52px, 9.5vw, 140px);
    color: #E35D5B;
    line-height: 1.05;
    letter-spacing: 0.12em;
  }

  /* ── Body ──
     Lower-left. Positioned so it doesn't overlap title.
     Top: 52vh ensures clear vertical separation even at large font sizes.
     Max-width ~42vw so it stays left of the photo/circle area.
  */
  .card-body {
    position: absolute;
    top: 52vh;
    left: 5vw;
    max-width: 52vw;
    font-family: 'Trispace', monospace;
    font-stretch: 112.5%;
    font-weight: 300;
    font-size: clamp(11px, 1vw, 16px);
    color: #3A2F2B;
    line-height: 1.85;
    letter-spacing: 0.02em;
    white-space: pre-line;
  }

  /* ── Text entrance animations ── */

  /* Forward navigation: text slides in from right */
  @keyframes slideInRight {
    from { opacity: 0; transform: translateX(60px); }
    to   { opacity: 1; transform: translateX(0); }
  }
  /* Backward navigation: text slides in from left */
  @keyframes slideInLeft {
    from { opacity: 0; transform: translateX(-60px); }
    to   { opacity: 1; transform: translateX(0); }
  }

  .enter-from-right {
    animation: slideInRight 0.65s cubic-bezier(0.16, 1, 0.3, 1) both;
  }
  .enter-from-left {
    animation: slideInLeft 0.65s cubic-bezier(0.16, 1, 0.3, 1) both;
  }

  /* ── Progress dots ── */
  .progress-dots {
    position: absolute;
    bottom: 3.5vh;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 0.6vw;
    z-index: 20;
    pointer-events: all;
  }
  .prog-dot {
    width: clamp(5px, 0.45vw, 9px);
    height: clamp(5px, 0.45vw, 9px);
    border-radius: 50%;
    border: none;
    background: #E35D5B;
    opacity: 0.3;
    cursor: pointer;
    padding: 0;
    transition: opacity 0.2s ease, transform 0.2s ease;
  }
  .prog-dot.active { opacity: 1; transform: scale(1.35); }
</style>
