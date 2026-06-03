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
  const CREDIT_LIMIT = 100000;
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

<svelte:head><title>Name on card – IOB</title></svelte:head>

<div class="screen">
  <StatusBar />

  <!-- BG lighting + Card section -->
  <div class="card-section">
    <!-- BG lighting -->
    <img src="/bg-lighting.svg" alt="" class="bg-lighting" draggable="false" aria-hidden="true" />

    <!-- Card with name overlay -->
    {#if showCard}
      <div class="card-scene" in:fly={{ y: 40, duration: 500, easing: cubicOut }}>
        <div class="card-wrap">
          <img src="{base}/tc-card.svg" alt="TC Travel Card" class="card-img" draggable="false" />
          <span class="card-name">{selectedName.toUpperCase()}</span>
        </div>
        <div class="card-shadow-wrap">
          <div class="card-shadow"></div>
        </div>
      </div>
    {/if}
  </div>

  <!-- Content section -->
  <div class="content-section">

    <!-- Heading -->
    {#if showHeading}
      <p class="heading-text" in:fade={{ duration: 350 }}>
        Great! You are one more step closer<br/>to your TC travel card
      </p>
    {/if}

    <!-- Credit limit -->
    {#if showLimit}
      <div class="limit-block" in:fly={{ y: 16, duration: 350, easing: cubicOut }}>
        <div class="limit-badge">
          <span class="limit-badge-text">Your approved credit limit</span>
        </div>
        <div class="limit-amount-wrap">
          <span class="limit-amount" class:shimmer={limitPopped}>{formattedLimit}</span>
        </div>
        <div class="limit-pill">
          <span class="limit-pill-text">Your limit can grow as you use the card</span>
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
    background: #FFFCF4;
    position: relative;
  }

  /* ── Card section ── */
  .card-section {
    position: relative;
    width: 100%;
    height: 355px;
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

  .card-scene {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    top: 91px;
    width: 288px;
    height: 230px;
    z-index: 1;
  }

  .card-wrap {
    width: 346px;
    height: 230px;
    position: absolute;
    top: 0;
    left: -29px;
    animation: cardFloat 3.5s ease-in-out infinite;
  }

  /* Shadow pinned to a fixed spot at the bottom — never moves vertically */
  .card-shadow-wrap {
    position: absolute;
    bottom: 0;
    left: 50%;
    transform: translateX(-50%);
  }

  .card-shadow {
    width: 160px;
    height: 10px;
    border-radius: 50%;
    background: rgba(0, 48, 129, 0.25);
    filter: blur(3px);
    animation: shadowScale 3.5s ease-in-out infinite;
  }

  /* Card floats up and down */
  @keyframes cardFloat {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-14px); }
  }

  /* Shadow: when card is up (50%) shadow shrinks, when card is down (0%/100%) shadow is full size */
  @keyframes shadowScale {
    0%, 100% { transform: scaleX(1); opacity: 0.7; }
    50% { transform: scaleX(0.55); opacity: 0.2; }
  }

  .card-img {
    width: 346px;
    height: 230px;
    display: block;
    user-select: none;
    pointer-events: none;
  }

  /* Name overlay on card — aligned near RuPay at bottom-left of the visible card */
  .card-name {
    position: absolute;
    left: 97px;
    top: 160px;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 700;
    font-size: 9px;
    line-height: 1.32;
    letter-spacing: 0.15em;
    color: rgba(255,255,255,0.9);
    pointer-events: none;
    user-select: none;
    transform: rotate(15.7deg);
    transform-origin: left center;
    z-index: 2;
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
    font-size: 32px;
    line-height: 1.32;
    color: #111827;
    text-align: center;
    position: relative;
    z-index: 0;
    text-shadow: 1px 0.5px 0px #FFFFFF;
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

  /* Limit increase pill */
  .limit-pill {
    display: flex;
    align-items: center;
    gap: 4px;
    padding: 4px 10px;
    background: #F0FDF4;
    border-radius: 20px;
    margin-top: 6px;
  }
  .limit-pill-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 500;
    font-size: 11px;
    color: #15803D;
    line-height: 1.4;
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
    background: #FFFFFF;
  }

  .name-item {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 12px;
    background: #FFFFFF;
    border: none;
    cursor: pointer;
    text-align: left;
    transition: background 0.15s;
  }
  .name-item:active { background: #FFFCF4; }

  .radio {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    border: 2px solid #D1D5DB;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    background: #FFFCF4;
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
    background: #FFFCF4;
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
