<!--
  ProgressComponent.svelte
  ========================
  Circular progress ring matching Figma node 1:9479.

  Props:
    step        — current step number (1-based)
    totalSteps  — total steps in the journey (default 20, so each step = 5%)

  The percent is derived as: Math.round((step / totalSteps) * 100)
  Clamped to 0–100.

  Usage:
    <ProgressComponent step={3} />
    <ProgressComponent step={7} totalSteps={20} />
-->
<script lang="ts">
  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';

  let {
    step = 1,
    totalSteps = 20
  }: {
    step?: number;
    totalSteps?: number;
  } = $props();

  // Derived percent — each step = 5% for a 20-step journey
  let targetPercent = $derived(
    Math.min(100, Math.max(0, Math.round((step / totalSteps) * 100)))
  );

  // Tweened value for smooth animation when step changes
  const animatedPercent = tweened(0, { duration: 900, easing: cubicOut });

  $effect(() => {
    animatedPercent.set(targetPercent);
  });

  // SVG geometry — bumped to 56px to contain 16px label
  const SIZE       = 56;
  const STROKE     = 5;
  const RADIUS     = (SIZE - STROKE) / 2;   // 25.5
  const CX         = SIZE / 2;              // 28
  const CY         = SIZE / 2;              // 28
  const CIRCUMFERENCE = 2 * Math.PI * RADIUS; // ~160.22

  const gradientId = 'prog-grad';

  // Dash offset driven by animated percent
  let dashOffset = $derived(
    CIRCUMFERENCE - ($animatedPercent / 100) * CIRCUMFERENCE
  );

  // Animated label — rounds the tweened value so it counts up
  let displayPercent = $derived(Math.round($animatedPercent));
</script>

<div class="progress-ring" aria-label="Progress: {targetPercent}%" role="progressbar"
  aria-valuenow={targetPercent} aria-valuemin={0} aria-valuemax={100}>

  <svg
    width={SIZE}
    height={SIZE}
    viewBox="0 0 {SIZE} {SIZE}"
    fill="none"
    overflow="visible"
  >
    <defs>
      <!-- Green gradient — Figma fill_2P5HHY: 128deg, #4ADE80 → #15803D
           128deg in SVG: x1=24.5% y1=100% x2=75.5% y2=0% -->
      <linearGradient id={gradientId} x1="24.5%" y1="100%" x2="75.5%" y2="0%" gradientUnits="objectBoundingBox">
        <stop offset="0%"   stop-color="#4ADE80" />
        <stop offset="100%" stop-color="#15803D" />
      </linearGradient>
    </defs>

    <!-- Track circle — Figma fill_C11ERF: #E5E7EB -->
    <circle
      cx={CX}
      cy={CY}
      r={RADIUS}
      stroke="#E5E7EB"
      stroke-width={STROKE}
      fill="none"
    />

    <!-- Progress arc — green gradient, starts from top (−90°) -->
    <circle
      cx={CX}
      cy={CY}
      r={RADIUS}
      stroke="url(#{gradientId})"
      stroke-width={STROKE}
      stroke-linecap="butt"
      fill="none"
      stroke-dasharray={CIRCUMFERENCE}
      stroke-dashoffset={dashOffset}
      transform="rotate(-90 {CX} {CY})"
    />
  </svg>

  <!-- Percentage label counts up with the arc -->
  <span class="label">{displayPercent}%</span>

</div>

<style>
  /* Outer container — 48×48, relative for absolute label */
  .progress-ring {
    position: relative;
    width: 56px;
    height: 56px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }

  /* Percentage label — Figma: Poppins 600 14px, fill_HEYGCS #111827
     positioned absolute, centered (matches Figma x:11 y:13 on 48px frame) */
  .label {
    position: absolute;
    font-family: 'Poppins', 'Manrope', sans-serif;
    font-weight: 600;
    font-size: 16px;
    line-height: 1;
    color: #111827;
    user-select: none;
    pointer-events: none;
    margin-top: 1px;
  }
</style>
