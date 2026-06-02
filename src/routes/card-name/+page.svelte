<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import { fly, fade } from 'svelte/transition';
  import { cubicOut } from 'svelte/easing';
  import { tweened } from 'svelte/motion';
  import { onMount } from 'svelte';
  import StatusBar from '$lib/components/StatusBar.svelte';

  // Name options from Aadhaar/PAN
  const nameOptions = ['Sajjad S', 'SS'];
  let selectedName = $state(nameOptions[0]);
  let loading = $state(false);

  // Animated credit limit counter
  const CREDIT_LIMIT = 22500;
  const animatedLimit = tweened(0, { duration: 1200, easing: cubicOut });
  let formattedLimit = $derived('₹' + Math.round($animatedLimit).toLocaleString('en-IN'));
  let limitPopped = $state(false);

  // Staggered load states
  let showCard = $state(false);
  let showHeading = $state(false);
  let showLimit = $state(false);
  let showNameSection = $state(false);

  onMount(() => {
    // Stagger each element in
    setTimeout(() => showCard = true, 100);
    setTimeout(() => showHeading = true, 400);
    setTimeout(() => { showLimit = true; animatedLimit.set(CREDIT_LIMIT).then(() => { limitPopped = true; }); }, 650);
    setTimeout(() => showNameSection = true, 900);
  });

  async function handleContinue() {
    loading = true;
    await new Promise(r => setTimeout(r, 600));
    loading = false;
    goto(`${base}/personal-details`);
  }
</script>

<svelte:head><title>Name on card – DCB</title></svelte:head>

<div class="screen">
  <StatusBar />

  <!-- BG lighting + Card section -->
  <div class="card-section">
    <!-- BG lighting -->
    <img src="/bg-lighting.svg" alt="" class="bg-lighting" draggable="false" aria-hidden="true" />

    <!-- Card with name overlay -->
    {#if showCard}
      <div class="card-wrap" in:fly={{ y: 40, duration: 500, easing: cubicOut }}>
        <img src="/dcb-card-updated.svg" alt="DCB Payless Card" class="card-img" draggable="false" />
        <!-- Name overlay on card -->
        <span class="card-name">{selectedName.toUpperCase()}</span>
      </div>
    {/if}
  </div>

  <!-- Content section -->
  <div class="content-section">

    <!-- Heading -->
    {#if showHeading}
      <p class="heading-text" in:fade={{ duration: 350 }}>
        Great! You are one more step closer<br/>to your DCB credit card
      </p>
    {/if}

    <!-- Credit limit -->
    {#if showLimit}
      <div class="limit-block" in:fly={{ y: 16, duration: 350, easing: cubicOut }}>
        <div class="limit-badge">
          <span class="limit-badge-text">Your credit limit</span>
        </div>
        <div class="limit-amount-wrap">
          <span class="limit-amount" class:shimmer={limitPopped}>{formattedLimit}</span>
        </div>
      </div>
    {/if}

    <!-- Name selection -->
    {#if showNameSection}
      <div class="name-section" in:fly={{ y: 16, duration: 350, easing: cubicOut }}>

        <!-- Divider with label -->
        <div class="divider-row">
          <div class="divider-line divider-line-left"></div>
          <span class="divider-label">Choose your preferred name on card</span>
          <div class="divider-line divider-line-right"></div>
        </div>

        <!-- Name options list -->
        <div class="name-list">
          {#each nameOptions as name, i}
            <button
              class="name-item"
              class:selected={selectedName === name}
              onclick={() => selectedName = name}
            >
              <!-- Radio -->
              <div class="radio" class:checked={selectedName === name}>
                {#if selectedName === name}
                  <div class="check-anim">
                    <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                      <circle cx="10" cy="10" r="10" fill="#15803D"/>
                      <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                  </div>
                {/if}
              </div>
              <span class="name-text">{name}</span>
            </button>
            {#if i < nameOptions.length - 1}
              <div class="name-divider"></div>
            {/if}
          {/each}
        </div>

      </div>
    {/if}

  </div>

  <div class="spacer"></div>

  <!-- Footer -->
  <div class="footer">
    <button
      class="btn-primary"
      disabled={loading}
      onclick={handleContinue}
    >
      {#if loading}
        <span class="spinner"></span>
      {:else}
        Continue my application
      {/if}
    </button>
  </div>

</div>

<style>
  .screen {
    width: 100%;
    min-height: 100dvh;
    display: flex;
    flex-direction: column;
    background: #FFFDF6;
    position: relative;
  }

  /* ── Card section ── */
  .card-section {
    position: relative;
    width: 100%;
    height: 283px;
    flex-shrink: 0;
    overflow: visible;
  }

  .bg-lighting {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    top: 52px;
    width: 325px;
    height: 322px;
    opacity: 0.79;
    pointer-events: none;
    user-select: none;
    transform-origin: center center;
    animation: bgRotate 30s linear infinite;
    z-index: 0;
  }

  @keyframes bgRotate {
    from { transform: translateX(-50%) rotate(0deg); }
    to   { transform: translateX(-50%) rotate(360deg); }
  }

  .card-wrap {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    top: 91px;
    width: 288px;
    height: 192px;
    z-index: 1;
  }

  .card-img {
    width: 288px;
    height: 192px;
    display: block;
    user-select: none;
    pointer-events: none;
  }

  /* Name overlay on card — matches Figma x:63 y:113, tilted to match card perspective */
  .card-name {
    position: absolute;
    left: 63px;
    top: 113px;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 10px;
    line-height: 1.32;
    letter-spacing: 0.2em;
    color: rgba(255,255,255,0.6);
    pointer-events: none;
    user-select: none;
    transform: rotate(9.09deg);
    transform-origin: left center;
    will-change: transform;
  }

  /* ── Content section ── */
  .content-section {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 24px;
    padding: 0 16px;
    flex-shrink: 0;
    position: relative;
    z-index: 1;
  }

  .heading-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 16px;
    line-height: 1.375;
    color: #111827;
    text-align: center;
    text-shadow: 1px 1px 0px rgba(0,0,0,0.2);
  }

  /* Credit limit */
  .limit-block {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2px;
    align-self: stretch;
  }

  .limit-badge {
    background: linear-gradient(90deg, #485563 0%, #29323C 100%);
    border: 1px solid #D1D5DB;
    border-radius: 20px;
    padding: 2px 8px;
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 10px;
  }
  .limit-badge-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 10px;
    line-height: 1.6;
    color: #FFFFFF;
    text-align: center;
  }

  .limit-amount-wrap {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
    border-radius: 4px;
  }

  .limit-amount {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 24px;
    line-height: 1.32;
    color: #111827;
    text-align: center;
    position: relative;
    z-index: 0;
  }

  /* CRED-style shimmer: two tilted white bars with 4px gap sweep left→right, once */
  .limit-amount.shimmer::after {
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
    animation: credShimmer 1.6s cubic-bezier(0.25, 0.1, 0.25, 1) forwards;
    pointer-events: none;
    z-index: 1;
  }

  @keyframes credShimmer {
    0%   { left: -80%; }
    100% { left: 150%; }
  }

  /* Name section */
  .name-section {
    display: flex;
    flex-direction: column;
    gap: 16px;
    width: calc(100% - 32px);
  }

  /* Divider with label */
  .divider-row {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 0 12px;
  }
  .divider-line {
    flex: 1;
    height: 0;
    border: none;
    border-top: 0.5px solid transparent;
  }
  .divider-line-left {
    border-image: linear-gradient(90deg, rgba(22,23,26,0) 0%, rgba(163,171,187,1) 100%) 1;
  }
  .divider-line-right {
    border-image: linear-gradient(90deg, rgba(163,171,187,1) 0%, rgba(22,23,26,0) 100%) 1;
  }
  .divider-label {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    line-height: 1.43;
    color: #6B7280;
    white-space: nowrap;
    text-align: center;
    flex-shrink: 0;
  }

  /* Name list */
  .name-list {
    display: flex;
    flex-direction: column;
    border: 1px solid #D1D5DB;
    border-radius: 8px;
    overflow: hidden;
  }

  .name-item {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 12px;
    background: #FFFDF6;
    border: none;
    cursor: pointer;
    text-align: left;
    transition: background 0.15s;
  }
  .name-item:active { background: #FFFDF6; }

  .radio {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    border: 2px solid #D1D5DB;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    background: #FFFDF6;
    transition: border-color 0.15s;
  }
  .radio.checked { border: none; background: none; }

  .check-anim {
    animation: checkPop 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
    display: flex;
  }
  @keyframes checkPop {
    0%   { transform: scale(0); opacity: 0; }
    60%  { transform: scale(1.15); opacity: 1; }
    100% { transform: scale(1); }
  }

  .name-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    line-height: 1.43;
    color: #111827;
    flex: 1;
  }

  .name-divider {
    height: 1px;
    background: #D1D5DB;
    margin: 0 12px;
  }

  /* ── Spacer ── */
  .spacer { flex: 1; }

  /* ── Footer ── */
  .footer { position: sticky; bottom: 0; z-index: 5; margin-top: auto;
    padding: 16px;
    background: #FFFDF6;
    flex-shrink: 0;
  }

  .btn-primary {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 48px;
    background: #184595;
    color: #FFFFFF;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 16px;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    transition: opacity 0.15s, transform 0.1s;
  }
  .btn-primary:active:not(:disabled) { opacity: 0.88; transform: scale(0.99); }
  .btn-primary:disabled { background: #D1D5DB; cursor: not-allowed; }

  .spinner {
    width: 20px; height: 20px;
    border: 2.5px solid rgba(255,255,255,0.4); border-top-color: #FFFFFF;
    border-radius: 50%; animation: spin 0.7s linear infinite;
  }
  @keyframes spin { to { transform: rotate(360deg); } }
</style>
