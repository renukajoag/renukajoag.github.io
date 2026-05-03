<script>
  import { onMount, onDestroy } from 'svelte';

  let canvas;
  let ctx;
  let animFrame;
  let t = 0;

  const BG = '#f2eded';

  // Orbs are kept well-separated so they never muddy each other.
  // Each lives in a different quadrant. Amplitudes are small enough
  // that they don't drift into their neighbours.
  // Radii are large (fill ~60-75% of their half-zone) but the low
  // alpha combined with 'multiply' keeps them vivid without blending grey.
  const orbs = [
    {
      // Green — upper-left quadrant
      r: 0.72, cx: -0.38, cy: -0.32,
      ar: 0.08, ax: 0.14, ay: 0.12,
      sx: 0.28, sy: 0.21, sr: 0.19,
      px: 0.0, py: 1.1, pr: 0.5,
      r0: '#8FAF9A', r1: '#5E9E82',
      inner: 0.78, mid: 0.48, midStop: 0.42, outerStop: 0.80,
    },
    {
      // Red/coral — upper-right quadrant
      r: 0.68, cx: 0.40, cy: -0.28,
      ar: 0.07, ax: 0.13, ay: 0.11,
      sx: 0.19, sy: 0.34, sr: 0.16,
      px: 2.1, py: 0.4, pr: 1.2,
      r0: '#E35D5B', r1: '#C43F3D',
      inner: 0.82, mid: 0.52, midStop: 0.40, outerStop: 0.78,
    },
    {
      // Smaller red — lower-left quadrant
      r: 0.52, cx: -0.38, cy: 0.36,
      ar: 0.06, ax: 0.12, ay: 0.10,
      sx: 0.38, sy: 0.25, sr: 0.30,
      px: 4.2, py: 2.8, pr: 3.1,
      r0: '#E35D5B', r1: '#C43F3D',
      inner: 0.80, mid: 0.50, midStop: 0.38, outerStop: 0.76,
    },
    {
      // Deeper green — lower-right quadrant
      r: 0.55, cx: 0.38, cy: 0.34,
      ar: 0.07, ax: 0.12, ay: 0.10,
      sx: 0.15, sy: 0.22, sr: 0.13,
      px: 1.5, py: 3.5, pr: 0.8,
      r0: '#6B9E8A', r1: '#4A8070',
      inner: 0.75, mid: 0.46, midStop: 0.40, outerStop: 0.78,
    },
  ];

  // ── Noise: two layers ──
  // Layer 1: coarse static grain (large tiles, baked once)
  // Layer 2: fine grain (small tiles)
  let noiseCoarse, noiseFine;

  function buildNoise() {
    noiseCoarse = makeNoiseCanvas(512, 38);
    noiseFine   = makeNoiseCanvas(128, 18);
  }

  function makeNoiseCanvas(size, alpha) {
    const nc = document.createElement('canvas');
    nc.width = nc.height = size;
    const nctx = nc.getContext('2d');
    const img = nctx.createImageData(size, size);
    for (let i = 0; i < img.data.length; i += 4) {
      const v = Math.random() * 255 | 0;
      img.data[i] = img.data[i+1] = img.data[i+2] = v;
      img.data[i+3] = alpha;
    }
    nctx.putImageData(img, 0, 0);
    return nc;
  }

  function hex(h) {
    return {
      r: parseInt(h.slice(1,3), 16),
      g: parseInt(h.slice(3,5), 16),
      b: parseInt(h.slice(5,7), 16),
    };
  }

  function draw() {
    if (!ctx) return;
    const W = canvas.width, H = canvas.height;

    // ── Base fill ──
    ctx.fillStyle = BG;
    ctx.fillRect(0, 0, W, H);

    // Zone covers the full canvas (orbs positioned by quadrant already)
    const zoneHW = W / 2;
    const zoneHH = H / 2;
    const cxC = W / 2, cyC = H / 2;

    // ── Draw orbs with 'multiply' — keeps colours pure on light bg ──
    ctx.save();
    ctx.globalCompositeOperation = 'multiply';

    for (const o of orbs) {
      const ox = cxC + (o.cx + Math.sin(t * o.sx + o.px) * o.ax) * zoneHW;
      const oy = cyC + (o.cy + Math.cos(t * o.sy + o.py) * o.ay) * zoneHH;
      const baseR = Math.min(zoneHW, zoneHH);
      const radius = baseR * (o.r + Math.sin(t * o.sr + o.pr) * o.ar);

      const c0 = hex(o.r0);
      const c1 = hex(o.r1);

      const grad = ctx.createRadialGradient(ox, oy, 0, ox, oy, radius);
      grad.addColorStop(0,          `rgba(${c0.r},${c0.g},${c0.b},${o.inner})`);
      grad.addColorStop(o.midStop,  `rgba(${c1.r},${c1.g},${c1.b},${o.mid})`);
      grad.addColorStop(o.outerStop,`rgba(${c0.r},${c0.g},${c0.b},0.08)`);
      grad.addColorStop(1.0,        `rgba(${c0.r},${c0.g},${c0.b},0.00)`);

      ctx.beginPath();
      drawBlob(ctx, ox, oy, radius, t * 0.35 + o.px, o);
      ctx.fillStyle = grad;
      ctx.fill();
    }

    ctx.restore();

    // ── Coarse grain (overlay) ──
    if (noiseCoarse) {
      ctx.save();
      ctx.globalAlpha = 0.22;
      ctx.globalCompositeOperation = 'overlay';
      const pat = ctx.createPattern(noiseCoarse, 'repeat');
      ctx.fillStyle = pat;
      ctx.fillRect(0, 0, W, H);
      ctx.restore();
    }

    // ── Fine grain (soft-light for extra grit) ──
    if (noiseFine) {
      ctx.save();
      ctx.globalAlpha = 0.14;
      ctx.globalCompositeOperation = 'soft-light';
      const pat = ctx.createPattern(noiseFine, 'repeat');
      ctx.fillStyle = pat;
      ctx.fillRect(0, 0, W, H);
      ctx.restore();
    }

    t += 0.007;
    animFrame = requestAnimationFrame(draw);
  }

  // Amorphous blob via layered harmonic perturbation + Catmull-Rom spline
  function drawBlob(ctx, cx, cy, r, phase, o) {
    const N = 10;
    const pts = [];
    for (let i = 0; i < N; i++) {
      const angle = (i / N) * Math.PI * 2;
      const perturb =
        0.10 * Math.sin(2 * angle + phase * 1.1) +
        0.07 * Math.cos(3 * angle - phase * 0.8 + o.py) +
        0.05 * Math.sin(5 * angle + phase * 1.5 + o.px) +
        0.03 * Math.cos(4 * angle - phase * 0.6);
      const rr = r * (1 + perturb);
      pts.push({ x: cx + Math.cos(angle) * rr, y: cy + Math.sin(angle) * rr });
    }
    ctx.beginPath();
    for (let i = 0; i < N; i++) {
      const p0 = pts[(i - 1 + N) % N];
      const p1 = pts[i];
      const p2 = pts[(i + 1) % N];
      const p3 = pts[(i + 2) % N];
      if (i === 0) ctx.moveTo(p1.x, p1.y);
      const tension = 0.45;
      ctx.bezierCurveTo(
        p1.x + (p2.x - p0.x) * tension / 3,
        p1.y + (p2.y - p0.y) * tension / 3,
        p2.x - (p3.x - p1.x) * tension / 3,
        p2.y - (p3.y - p1.y) * tension / 3,
        p2.x, p2.y
      );
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
    buildNoise();
    resize();
    draw();
    window.addEventListener('resize', resize);
  });

  onDestroy(() => {
    cancelAnimationFrame(animFrame);
    window.removeEventListener('resize', resize);
  });
</script>

<canvas bind:this={canvas} class="bg-canvas"></canvas>

<style>
  .bg-canvas {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    display: block;
  }
</style>
