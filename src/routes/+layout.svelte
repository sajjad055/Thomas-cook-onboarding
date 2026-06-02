<script lang="ts">
  import '../app.css';
  import { navigating } from '$app/stores';
  import { fly, fade, scale } from 'svelte/transition';
  import { cubicOut } from 'svelte/easing';
  import { page } from '$app/stores';

  let { children } = $props();

  // Routes where the page fades in (entering) and fades out (leaving)
  const fadeRoutes = new Set(['/payment/success']);

  // Track navigation direction based on route order
  const routeOrder: Record<string, number> = {
    '/existing-customer': 1,
    '/mobile': 2,
    '/mobile/otp': 3,
    '/aadhaar': 4,
    '/aadhaar/otp': 4.5,
    '/pan': 5,
    '/fd': 6,
    '/payment': 7,
    '/payment/success': 7.5,
    '/card-name': 8,
    '/personal-details': 9,
    '/income': 10,
    '/address': 11,
    '/vkyc': 12,
    '/card-activation': 13,
  };

  let prevPath = $state('');
  let direction = $state(1);
  // Whether the CURRENT page being rendered is a fade route
  let currentIsFade = $derived(fadeRoutes.has($page.url.pathname));
  // Whether we're LEAVING a fade route (prev was fade)
  let leavingFade = $state(false);

  $effect(() => {
    const current = $page.url.pathname;
    const prev = prevPath;
    if (prev && current !== prev) {
      const currentOrder = routeOrder[current] ?? 99;
      const prevOrder = routeOrder[prev] ?? 99;
      direction = currentOrder >= prevOrder ? 1 : -1;
      leavingFade = fadeRoutes.has(prev);
    }
    prevPath = current;
  });
</script>

<!-- Outer shell stays fixed; only inner content transitions -->
<div class="layout-shell">
  {#key $page.url.pathname}
    <div
      class="page-wrapper"
      in:fly={currentIsFade
        ? { x: 0, y: 0, duration: 400, delay: 250, opacity: 0 }
        : { x: direction * 360, duration: 450, easing: cubicOut, delay: leavingFade ? 280 : 40 }}
      out:fade={currentIsFade
        ? { duration: 250 }
        : { duration: leavingFade ? 250 : 0 }}
    >
      {@render children()}
    </div>
  {/key}
</div>

<style>
  .layout-shell {
    width: 100%;
    min-height: 100dvh;
    position: relative;
    overflow: hidden;
    margin: 0 auto;
  }

  .page-wrapper {
    width: 100%;
    min-height: 100dvh;
    position: absolute;
    top: 0;
    left: 0;
    will-change: transform;
  }

</style>
