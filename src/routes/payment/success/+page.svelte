<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';
  import { browser } from '$app/environment';
  import StatusBar from '$lib/components/StatusBar.svelte';

  let lottieReady = $state(false);
  let showReceipt = $state(false);
  let showNote = $state(false);

  onMount(async () => {
    if (browser) {
      try {
        await import('@lottiefiles/dotlottie-wc');
      } catch (e) {
        console.warn('Lottie load failed', e);
      }
    }
    lottieReady = true;

    setTimeout(() => showReceipt = true, 1600);
    setTimeout(() => showNote = true, 2200);
    setTimeout(() => goto(`${base}/card-name`), 4200);
  });

  const now = new Date();
  const dateStr = now.toLocaleDateString('en-IN', {
    day: 'numeric',
    month: 'long',
    year: 'numeric',
  });
  const timeStr = now.toLocaleTimeString('en-IN', {
    hour: 'numeric',
    minute: '2-digit',
    hour12: true,
  });
</script>

<svelte:head><title>Payment successful – DCB</title></svelte:head>

<div class="screen">
  <StatusBar />

  <div class="center-area">

    <!-- Lottie success animation -->
    {#if lottieReady}
      <div class="lottie-wrap">
        <dotlottie-wc
          src="/success.lottie"
          autoplay
          speed="0.7"
          style="width: 120px; height: 120px;"
        ></dotlottie-wc>
      </div>
    {/if}

    <!-- Receipt card — fades in as one block -->
    {#if showReceipt}
      <div class="receipt-card" in:fade={{ duration: 500 }}>
        <p class="amount">₹25,000</p>
        <p class="status-text">FD payment successful</p>
        <div class="date-block">
          <div class="divider"></div>
          <p class="date-text">{dateStr}, {timeStr}</p>
        </div>
        <div class="scallop-edge" aria-hidden="true">
          {#each Array(30) as _}
            <div class="scallop-dot"></div>
          {/each}
        </div>
      </div>
    {/if}

    <!-- Refund note -->
    {#if showNote}
      <p class="refund-note" in:fade={{ duration: 400 }}>
        This FD amount will be refunded to your source account if you don't complete the remaining application steps within 72 hours.
      </p>
    {/if}

  </div>
</div>

<style>
  .screen {
    width: 100%;
    min-height: 100dvh;
    display: flex;
    flex-direction: column;
    background: #FFFCF4;
  }

  .center-area {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 24px;
    padding: 0 16px;
  }

  .lottie-wrap {
    width: 120px;
    height: 120px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .receipt-card {
    background: #FFFCF4;
    width: calc(100% - 32px);
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 12px;
    padding: 24px 0 20px;
    position: relative;
    border-radius: 16px 16px 0 0;
  }

  .amount {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 24px;
    line-height: 132%;
    color: #111827;
    text-align: center;
  }

  .status-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 18px;
    color: #6B7280;
    text-align: center;
  }

  .date-block {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 8px;
    width: 100%;
    padding: 4px 12px 0;
  }

  .divider {
    width: 173px;
    height: 0;
    border-top: 0.5px dashed #D1D5DB;
  }

  .date-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 12px;
    line-height: 18px;
    color: #111827;
    text-align: center;
  }

  .scallop-edge {
    position: absolute;
    bottom: -6px;
    left: 0;
    right: 0;
    display: flex;
    justify-content: center;
    gap: 0;
    overflow: hidden;
  }

  .scallop-dot {
    width: 11px;
    height: 7px;
    background: #FFFCF4;
    border-radius: 50% 50% 0 0;
    flex-shrink: 0;
  }

  .refund-note {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 10px;
    line-height: 16px;
    color: #6B7280;
    text-align: center;
    max-width: 278px;
  }
</style>
