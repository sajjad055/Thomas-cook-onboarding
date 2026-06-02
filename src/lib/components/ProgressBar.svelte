<!--
  ProgressBar.svelte
  ==================
  Horizontal progress pill showing application completion.
  Text over the green fill is white; text outside is dark.

  Props:
    step        — current step (1-based)
    totalSteps  — total steps (default 20)

  Usage:
    <ProgressBar step={14} />
-->
<script lang="ts">
  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';

  let {
    step = 1,
    totalSteps = 20,
  }: {
    step?: number;
    totalSteps?: number;
  } = $props();

  let targetPercent = $derived(
    Math.min(100, Math.max(0, Math.round((step / totalSteps) * 100)))
  );

  const animatedPercent = tweened(0, { duration: 900, easing: cubicOut });

  $effect(() => {
    animatedPercent.set(targetPercent);
  });

  let displayPercent = $derived(Math.round($animatedPercent));
  let fillWidth = $derived(`${$animatedPercent}%`);
</script>

<div class="progress-bar-wrap">
  <!-- Percentage number -->
  <span class="progress-percent" style="color: #00676B">{displayPercent}%</span>

  <!-- Pill -->
  <div class="progress-pill">
    <!-- Green fill bar -->
    <div class="progress-fill" style="width: {fillWidth}"></div>

    <!-- Dark text (base layer — visible outside the fill) -->
    <span class="progress-label progress-label-dark">Application progress</span>

    <!-- White text (clipped to the fill width) -->
    <span class="progress-label progress-label-light" style="clip-path: inset(0 {100 - $animatedPercent}% 0 0)">
      Application progress
    </span>
  </div>
</div>

<style>
  .progress-bar-wrap {
    display: flex;
    align-items: center;
    gap: 2px;
    flex-shrink: 0;
  }

  .progress-percent {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 20px;
    line-height: 28px;
    color: #00676B;
    user-select: none;
  }

  .progress-pill {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 4px 8px;
    background: #F7F7F7;
    border: 0.5px solid #E1E6EC;
    border-radius: 8px;
    overflow: hidden;
    min-width: 120px;
  }

  /* Green gradient fill — absolute, left-aligned */
  .progress-fill {
    position: absolute;
    top: -1px;
    left: 0;
    bottom: -1px;
    background: linear-gradient(0deg, #00676B 50%, #00C9D1 100%);
    border-radius: 8px 0 0 8px;
    transition: width 0.1s linear;
    z-index: 1;
  }

  /* Base text styles */
  .progress-label {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 130%;
    white-space: nowrap;
    user-select: none;
    position: relative;
  }

  /* Dark text — sits behind, visible where there's no fill */
  .progress-label-dark {
    color: #111827;
    z-index: 2;
  }

  /* White text — sits on top, clipped to the fill width */
  .progress-label-light {
    color: #FFFFFF;
    position: absolute;
    left: 8px;
    z-index: 3;
  }
</style>
