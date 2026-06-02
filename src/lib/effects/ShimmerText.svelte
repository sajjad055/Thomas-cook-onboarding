<!--
  ShimmerText.svelte
  ==================
  CRED-style double shimmer effect — two solid white tilted bars sweep
  left→right over slotted text, once, when `trigger` flips to true.

  Props:
    trigger   — set to true to fire the shimmer (one-shot)
    duration  — sweep duration in ms (default 1600)
    class     — optional extra class on the wrapper

  Usage:
    <ShimmerText trigger={limitPopped}>
      ₹22,500
    </ShimmerText>

  The parent must set a non-transparent background on the wrapper or the
  white bars won't be visible. The wrapper clips overflow so the bars
  don't bleed outside the text bounds.
-->
<script lang="ts">
  let {
    trigger = false,
    duration = 1600,
    loop = false,
    class: extraClass = '',
    children,
  }: {
    trigger?: boolean;
    duration?: number;
    loop?: boolean;
    class?: string;
    children?: any;
  } = $props();
</script>

<span
  class="shimmer-wrap {extraClass}"
  class:shimmer={trigger}
  style="--shimmer-duration: {loop ? duration * 3 : duration}ms; --shimmer-iteration: {loop ? 'infinite' : '1'}"
>
  {@render children?.()}
</span>

<style>
  .shimmer-wrap {
    position: relative;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
    border-radius: inherit;
  }

  /* Two solid white tilted bars with a ~4px gap, sweep left→right once */
  .shimmer-wrap.shimmer::after {
    content: '';
    position: absolute;
    top: -20%;
    left: -80%;
    width: 60%;
    height: 140%;
    background: linear-gradient(
      to right,
      rgba(255,255,255,0)   0%,
      rgba(255,255,255,0)   28%,
      rgba(255,255,255,1)   28%,
      rgba(255,255,255,1)   44%,
      rgba(255,255,255,0)   44%,
      rgba(255,255,255,0)   52%,
      rgba(255,255,255,1)   52%,
      rgba(255,255,255,1)   68%,
      rgba(255,255,255,0)   68%,
      rgba(255,255,255,0)   100%
    );
    transform: skewX(-28deg);
    animation: credShimmer var(--shimmer-duration) cubic-bezier(0.25, 0.1, 0.25, 1) var(--shimmer-iteration);
    pointer-events: none;
    z-index: 1;
  }

  @keyframes credShimmer {
    0%   { left: -80%; }
    35%  { left: 150%; }
    100% { left: 150%; }
  }
</style>
