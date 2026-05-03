<script>
  import { onMount, onDestroy } from 'svelte';

  export let onBack = () => {};

  let canvas;
  let ctx;
  let animFrame;
  let t = 0;

  // ── Orb definitions ──
  // Each orb has a base position (cx, cy as fraction of 83% central zone),
  // a base radius, movement amplitude, speed, and phase offsets.
  // They drift around the center of the page within the 83% zone.
  const orbs = [
    {
      // Large sage green orb — dominant background presence
      r:      0.78,   // radius as fraction of zone half-size
      cx:     0.0,    // offset from zone center (fraction of zone half-size)
      cy:    -0.08,
      ar:     0.12,   // amplitude for radius oscillation
      ax:     0.22,   // amplitude for x drift
      ay:     0.18,   // amplitude for y drift
      sx:     0.31,   // speed x
      sy:     0.19,   // speed y
      sr:     0.23,   // speed radius
      px:     0.0,    // phase x
      py:     1.1,    // phase y
      pr:     0.5,    // phase radius
      // color: sage green glow
      r0: '#8FAF9A',  // inner
      r1: '#6B9E8A',  // mid
      r1s: 0.5,       // mid stop
      r2: 'transparent',
    },
    {
      // Medium-large red/coral orb — upper right
      r:      0.68,
      cx:     0.28,
      cy:    -0.22,
      ar:     0.09,
      ax:     0.20,
      ay:     0.16,
      sx:     0.22,
      sy:     0.37,
      sr:     0.18,
      px:     2.1,
      py:     0.4,
      pr:     1.2,
      r0: '#E35D5B',
      r1: '#C94B49',
      r1s: 0.45,
      r2: 'transparent',
    },
    {
      // Smaller red orb — lower left, moves more
      r:      0.48,
      cx:    -0.30,
      cy:     0.28,
      ar:     0.07,
      ax:     0.26,
      ay:     0.22,
      sx:     0.41,
      sy:     0.28,
      sr:     0.33,
      px:     4.2,
      py:     2.8,
      pr:     3.1,
      r0: '#E35D5B',
      r1: '#C94B49',
      r1s: 0.4,
      r2: 'transparent',
    },
    {
      // Subtle green accent — lower right, slow
      r:      0.40,
      cx:     0.25,
      cy:     0.30,
      ar:     0.08,
      ax:     0.18,
      ay:     0.14,
      sx:     0.17,
      sy:     0.24,
      sr:     0.14,
      px:     1.5,
      py:     3.5,
      pr:     0.8,
      r0: '#7d9e8a',
      r1: '#5a8070',
      r1s: 0.5,
      r2: 'transparent',
    },
  ];

  // Noise texture (generated once via offscreen canvas)
  let noiseCanvas;
  let noiseCtx;
  const NOISE_SIZE = 256;

  function buildNoiseTexture() {
    noiseCanvas = document.createElement('canvas');
    noiseCanvas.width  = NOISE_SIZE;
    noiseCanvas.height = NOISE_SIZE;
    noiseCtx = noiseCanvas.getContext('2d');
    const imgData = noiseCtx.createImageData(NOISE_SIZE, NOISE_SIZE);
    const d = imgData.data;
    for (let i = 0; i < d.length; i += 4) {
      const v = Math.random() * 255 | 0;
      d[i] = d[i+1] = d[i+2] = v;
      d[i+3] = 28; // very subtle alpha
    }
    noiseCtx.putImageData(imgData, 0, 0);
  }

  function hexToRgb(hex) {
    const r = parseInt(hex.slice(1,3), 16);
    const g = parseInt(hex.slice(3,5), 16);
    const b = parseInt(hex.slice(5,7), 16);
    return { r, g, b };
  }

  function draw() {
    if (!ctx) return;
    const W = canvas.width;
    const H = canvas.height;

    ctx.clearRect(0, 0, W, H);

    // Background — near-black
    ctx.fillStyle = '#0a0a0a';
    ctx.fillRect(0, 0, W, H);

    // 83% zone: half-sizes
    const zoneHW = (W * 0.83) / 2;
    const zoneHH = (H * 0.83) / 2;
    const zoneCX = W / 2;
    const zoneCY = H / 2;

    // Draw each orb
    ctx.save();

    // Soft compositing for bloom
    ctx.globalCompositeOperation = 'screen';

    for (const o of orbs) {
      // Compute animated position and radius
      const ox = zoneCX + (o.cx + Math.sin(t * o.sx + o.px) * o.ax) * zoneHW;
      const oy = zoneCY + (o.cy + Math.cos(t * o.sy + o.py) * o.ay) * zoneHH;
      const baseR = Math.min(zoneHW, zoneHH);
      const radius = baseR * (o.r + Math.sin(t * o.sr + o.pr) * o.ar);

      // Build radial gradient
      const grad = ctx.createRadialGradient(ox, oy, 0, ox, oy, radius);

      // Inner colour
      const inner = hexToRgb(o.r0);
      grad.addColorStop(0,   `rgba(${inner.r},${inner.g},${inner.b},0.90)`);

      // Mid
      const mid = hexToRgb(o.r1);
      grad.addColorStop(o.r1s, `rgba(${mid.r},${mid.g},${mid.b},0.55)`);

      // Outer — fade to transparent
      grad.addColorStop(0.78, `rgba(${inner.r},${inner.g},${inner.b},0.12)`);
      grad.addColorStop(1.0,  `rgba(${inner.r},${inner.g},${inner.b},0.00)`);

      ctx.beginPath();
      // Amorphous blob shape via bezier — we morph the control points with time
      drawBlob(ctx, ox, oy, radius, t * 0.4 + o.px, o);
      ctx.fillStyle = grad;
      ctx.fill();
    }

    ctx.restore();

    // Overlay noise texture for grain
    if (noiseCanvas) {
      ctx.save();
      ctx.globalAlpha = 0.18;
      ctx.globalCompositeOperation = 'overlay';
      // Tile the noise
      const pat = ctx.createPattern(noiseCanvas, 'repeat');
      ctx.fillStyle = pat;
      ctx.fillRect(0, 0, W, H);
      ctx.restore();
    }

    // Vignette — dark edges
    const vign = ctx.createRadialGradient(W/2, H/2, Math.min(W,H)*0.28, W/2, H/2, Math.max(W,H)*0.75);
    vign.addColorStop(0,   'rgba(0,0,0,0)');
    vign.addColorStop(0.6, 'rgba(0,0,0,0.1)');
    vign.addColorStop(1,   'rgba(0,0,0,0.72)');
    ctx.fillStyle = vign;
    ctx.fillRect(0, 0, W, H);

    t += 0.008;
    animFrame = requestAnimationFrame(draw);
  }

  /**
   * Draw an amorphous closed blob around (cx, cy) with approximate radius r.
   * Deformation is driven by t (time) and orb phase.
   * We use 8 evenly-spaced points, each perturbed radially with sin/cos noise,
   * connected by smooth bezier curves.
   */
  function drawBlob(ctx, cx, cy, r, phase, o) {
    const N = 8;
    const pts = [];

    for (let i = 0; i < N; i++) {
      const angle = (i / N) * Math.PI * 2;
      // Radial perturbation — layered harmonics for organic shape
      const perturb =
        0.12 * Math.sin(2 * angle + phase * 1.1) +
        0.08 * Math.cos(3 * angle - phase * 0.7 + o.py) +
        0.05 * Math.sin(5 * angle + phase * 1.4 + o.px) +
        0.04 * Math.cos(angle * 4 - phase * 0.9);

      const rr = r * (1 + perturb);
      pts.push({
        x: cx + Math.cos(angle) * rr,
        y: cy + Math.sin(angle) * rr,
      });
    }

    // Draw closed smooth curve through pts using catmull-rom → bezier
    ctx.beginPath();
    for (let i = 0; i < N; i++) {
      const p0 = pts[(i - 1 + N) % N];
      const p1 = pts[i];
      const p2 = pts[(i + 1) % N];
      const p3 = pts[(i + 2) % N];

      if (i === 0) ctx.moveTo(p1.x, p1.y);

      // Catmull-Rom to Bezier conversion
      const tension = 0.5;
      const cp1x = p1.x + (p2.x - p0.x) * tension / 3;
      const cp1y = p1.y + (p2.y - p0.y) * tension / 3;
      const cp2x = p2.x - (p3.x - p1.x) * tension / 3;
      const cp2y = p2.y - (p3.y - p1.y) * tension / 3;

      ctx.bezierCurveTo(cp1x, cp1y, cp2x, cp2y, p2.x, p2.y);
    }
    ctx.closePath();
  }

  function resize() {
    if (!canvas) return;
    canvas.width  = canvas.offsetWidth;
    canvas.height = canvas.offsetHeight;
  }

  onMount(() => {
    ctx = canvas.getContext('2d');
    buildNoiseTexture();
    resize();
    draw();
    window.addEventListener('resize', resize);
  });

  onDestroy(() => {
    cancelAnimationFrame(animFrame);
    window.removeEventListener('resize', resize);
  });
</script>

<div class="work-root">
  <canvas bind:this={canvas} class="orb-canvas"></canvas>

  <!-- Scroll-up hint -->
  <p class="back-hint">scroll up to go back</p>
</div>

<style>
  .work-root {
    position: absolute;
    inset: 0;
    z-index: 20;
    overflow: hidden;
    background: #0a0a0a;
  }

  .orb-canvas {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    display: block;
  }

  .back-hint {
    position: absolute;
    bottom: 4vh;
    left: 50%;
    transform: translateX(-50%);
    font-family: 'Trispace', monospace;
    font-stretch: 112.5%;
    font-weight: 300;
    font-size: clamp(9px, 0.8vw, 14px);
    color: rgba(255,255,255,0.25);
    letter-spacing: 0.18em;
    pointer-events: none;
    animation: fadeIn 1.5s ease 1s both;
  }

  @keyframes fadeIn {
    from { opacity: 0; }
    to   { opacity: 1; }
  }
</style>
