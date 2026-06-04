<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';

  function handleStart() {
    goto(`${base}/mobile`);
  }
</script>

<svelte:head><title>TC Travel Card</title></svelte:head>

<div class="screen" onclick={handleStart} role="button" tabindex="0" onkeydown={(e) => { if (e.key === 'Enter') handleStart(); }}>
  <img src="{base}/thomascook-landing.svg" alt="TC Travel Card - Get Card in 10 min" class="landing-svg" />
</div>

<style>
  .screen {
    width: 100%;
    height: 100dvh;
    display: flex;
    flex-direction: column;
    background: #FFFCF4;
    cursor: pointer;
    overflow-y: scroll;
    -webkit-overflow-scrolling: touch;
  }

  .landing-svg {
    width: 100%;
    height: auto;
    display: block;
    flex-shrink: 0;
  }
</style>
