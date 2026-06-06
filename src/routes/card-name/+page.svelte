<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import { fly, fade } from 'svelte/transition';
  import { cubicOut } from 'svelte/easing';
  import { tweened } from 'svelte/motion';
  import { onMount } from 'svelte';
  import StatusBar from '$lib/components/StatusBar.svelte';
  import BottomSheet from '$lib/components/BottomSheet.svelte';

  // Name options from Aadhaar/PAN
  const nameOptions = ['Sajjad S', 'SS'];
  let selectedName = $state(nameOptions[0]);
  let loading = $state(false);

  // Animated credit limit counter
  const APPROVED_LIMIT = 100000;
  const MIN_LIMIT = 25000;
  let selectedLimit = $state(APPROVED_LIMIT);
  const animatedLimit = tweened(0, { duration: 1200, easing: cubicOut });
  let formattedLimit = $derived('₹' + Math.round($animatedLimit).toLocaleString('en-IN'));
  let displayLimit = $derived('₹' + selectedLimit.toLocaleString('en-IN'));
  let limitPopped = $state(false);

  // Edit limit bottom sheet
  let showEditLimit = $state(false);
  let tempLimit = $state(APPROVED_LIMIT);
  const animatedTempLimit = tweened(APPROVED_LIMIT, { duration: 150, easing: cubicOut });
  let sliderPercent = $derived(((tempLimit - MIN_LIMIT) / (APPROVED_LIMIT - MIN_LIMIT)) * 100);
  let formattedTempLimit = $derived('₹' + Math.round($animatedTempLimit).toLocaleString('en-IN'));

  function openEditLimit() {
    tempLimit = selectedLimit;
    animatedTempLimit.set(selectedLimit, { duration: 0 });
    showEditLimit = true;
  }

  function handleSliderInput(e: Event) {
    const input = e.target as HTMLInputElement;
    const percent = parseFloat(input.value);
    let newLimit = Math.round(MIN_LIMIT + (percent / 100) * (APPROVED_LIMIT - MIN_LIMIT));
    // Round to nearest 1000
    newLimit = Math.round(newLimit / 1000) * 1000;
    tempLimit = newLimit;
    animatedTempLimit.set(newLimit);
  }

  function confirmLimit() {
    selectedLimit = tempLimit;
    animatedLimit.set(selectedLimit);
    showEditLimit = false;
  }

  function continueWithMax() {
    selectedLimit = APPROVED_LIMIT;
    tempLimit = APPROVED_LIMIT;
    animatedLimit.set(APPROVED_LIMIT);
    showEditLimit = false;
  }

  // Staggered load states
  let showCard = $state(false);
  let showHeading = $state(false);
  let showLimit = $state(false);
  let showNameSection = $state(false);

  // Declarations
  let showDeclarations = $state(false);
  let declarations = $state([false, false, false, false]);
  let allChecked = $derived(declarations.every(d => d));

  const declarationTexts = [
    'I have read and understood the MITC & Key Fact Statement (KFS) for this credit card product.',
    'I accept the terms and conditions of IOB Bank applicable to the TC Travel Card.',
    'I acknowledge that Thomas Cook is the co-brand partner for this card, issued in partnership with IOB Bank.',
    'I consent to sharing my information with IOB Bank and Thomas Cook for the purpose of card issuance and servicing.',
  ];

  function toggleDeclaration(i: number) {
    declarations[i] = !declarations[i];
  }

  function acceptAll() {
    declarations = [true, true, true, true];
  }

  async function handleContinueAfterDeclarations() {
    loading = true;
    await new Promise(r => setTimeout(r, 600));
    loading = false;
    showDeclarations = false;
    goto(`${base}/personal-details`);
  }

  onMount(() => {
    setTimeout(() => showCard = true, 100);
    setTimeout(() => showHeading = true, 400);
    setTimeout(() => { showLimit = true; animatedLimit.set(selectedLimit).then(() => { limitPopped = true; }); }, 650);
    setTimeout(() => showNameSection = true, 900);
  });

  function handleContinue() {
    showDeclarations = true;
  }
</script>

<svelte:head><title>Name on card – IOB</title></svelte:head>

<div class="screen">
  <StatusBar />

  <!-- BG lighting + Card section -->
  <div class="card-section">
    <!-- BG lighting -->
    <img src="{base}/bg-lighting.svg" alt="" class="bg-lighting" draggable="false" aria-hidden="true" />

    <!-- Card with name overlay -->
    {#if showCard}
      <div class="card-scene" in:fly={{ y: 40, duration: 500, easing: cubicOut }}>
        <div class="card-wrap">
          <img src="{base}/tc-card-bank.svg" alt="TC Travel Card" class="card-img" draggable="false" />
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
          <button class="edit-limit-btn" onclick={openEditLimit} aria-label="Edit credit limit">
            <i class="ph ph-pencil-simple" style="font-size:24px; color:#111827"></i>
          </button>
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

  <!-- Bank Partner Logos -->
  <div class="partner-logos">
    <img src="{base}/logos-bank-partner.svg" alt="Bank Partners" class="partner-img" draggable="false" />
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

<!-- Declarations Bottom Sheet -->
<BottomSheet bind:open={showDeclarations} title="Accept terms and conditions">
  <div class="declarations-list">
    {#each declarationTexts as text, i}
      <button class="declaration-item" onclick={() => toggleDeclaration(i)}>
        <div class="checkbox" class:checked={declarations[i]}>
          {#if declarations[i]}
            <div class="check-anim">
              <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
                <rect width="18" height="18" rx="4" fill="#184595"/>
                <path d="M5.5 9.2L7.8 11.6L12.5 6.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </div>
          {:else}
            <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
              <rect x="0.5" y="0.5" width="17" height="17" rx="3.5" stroke="#D1D5DB"/>
            </svg>
          {/if}
        </div>
        <span class="declaration-text">{text}</span>
      </button>
    {/each}
  </div>

  {#snippet footer()}
    {#if allChecked}
      <button class="btn-primary-sheet" onclick={handleContinueAfterDeclarations} disabled={loading}>
        {#if loading}<span class="spinner"></span>{:else}Continue{/if}
      </button>
    {:else}
      <button class="btn-secondary-sheet" onclick={acceptAll}>
        Accept All
      </button>
    {/if}
  {/snippet}
</BottomSheet>

<!-- Edit Limit Bottom Sheet -->
<BottomSheet bind:open={showEditLimit} title="Adjust your credit limit">
  <div class="edit-limit-content">
    <!-- Combined card with slider and info -->
    <div class="limit-adjust-card">
      <!-- Slider section -->
      <div class="slider-section">
        <div class="slider-value-display">
          <span class="slider-value-amount">{formattedTempLimit}</span>
        </div>
        
        <!-- shadcn-style slider -->
        <div class="shadcn-slider" role="slider" aria-valuemin={MIN_LIMIT} aria-valuemax={APPROVED_LIMIT} aria-valuenow={tempLimit}>
          <div class="slider-track">
            <div class="slider-range" style="width: {sliderPercent}%"></div>
          </div>
          <input 
            type="range" 
            min="0" 
            max="100" 
            value={sliderPercent}
            oninput={handleSliderInput}
            class="slider-input"
          />
          <div class="slider-thumb" style="left: calc({sliderPercent}% - 10px)"></div>
        </div>
      </div>

      <!-- Divider -->
      <div class="limit-card-divider"></div>

      <!-- Info section -->
      <div class="limit-info-section">
        <img src="{base}/cash-stack.svg" alt="" class="cash-stack-icon" />
        <p class="limit-info-text">Even if you choose a lower limit now, you can instantly upgrade up to your approved limit anytime</p>
      </div>
    </div>
  </div>

  {#snippet footer()}
    <div class="edit-limit-footer">
      <button class="btn-primary-sheet" onclick={confirmLimit}>
        Confirm limit
      </button>
      <button class="btn-text-link" onclick={continueWithMax}>
        Continue with maximum limit
      </button>
    </div>
  {/snippet}
</BottomSheet>

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
    height: clamp(280px, 45vh, 355px);
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
    left: 101px;
    top: 142.5px;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 700;
    font-size: 7px;
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
    gap: 8px;
    overflow: hidden;
    border-radius: 4px;
  }

  .edit-limit-btn {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 32px;
    height: 32px;
    background: transparent;
    border: none;
    cursor: pointer;
    transition: opacity 0.15s;
  }
  .edit-limit-btn:active { opacity: 0.6; }

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
    background: transparent;
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
    padding: 16px 16px calc(24px + env(safe-area-inset-bottom));
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

  /* Declarations */
  .declarations-list {
    display: flex;
    flex-direction: column;
    background: #FFFFFF;
    border-radius: 12px;
    border: 1px solid #F3F4F6;
    overflow: hidden;
  }

  .declaration-item {
    display: flex;
    align-items: flex-start;
    gap: 10px;
    padding: 14px 16px;
    background: none;
    border: none;
    border-bottom: 1px solid #F3F4F6;
    cursor: pointer;
    text-align: left;
  }
  .declaration-item:last-child {
    border-bottom: none;
  }

  .checkbox {
    width: 18px;
    height: 18px;
    flex-shrink: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-top: 1px;
  }

  .declaration-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 1.5;
    color: #111827;
    flex: 1;
  }

  .btn-primary-sheet {
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
    box-shadow: 0px 4px 0px #06142A;
    transition: opacity 0.15s, transform 0.1s;
  }
  .btn-primary-sheet:active:not(:disabled) { opacity: 0.88; transform: scale(0.99); }
  .btn-primary-sheet:disabled { background: #D1D5DB; box-shadow: none; cursor: not-allowed; }

  .btn-secondary-sheet {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 48px;
    background: transparent;
    color: #184595;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 16px;
    border: 1.5px solid #184595;
    border-radius: 8px;
    cursor: pointer;
    transition: opacity 0.15s;
  }
  .btn-secondary-sheet:active { opacity: 0.85; }

  /* Partner logos */
  .partner-logos {
    display: flex;
    justify-content: center;
    margin-top: 24px;
    flex-shrink: 0;
  }
  .partner-img {
    height: 46px;
    filter: grayscale(100%);
    width: auto;
    user-select: none;
    pointer-events: none;
  }

  /* Edit Limit Bottom Sheet */
  .edit-limit-content {
    display: flex;
    flex-direction: column;
  }

  .edit-limit-subtitle {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 500;
    font-size: 14px;
    line-height: 1.5;
    color: #6B7280;
    text-align: center;
  }

  .slider-card {
    background: #FFFFFF;
    border-radius: 12px;
    padding: 16px 20px;
  }

  .limit-adjust-card {
    background: #FFFFFF;
    border-radius: 12px;
    border: 1px solid #E5E7EB;
    overflow: hidden;
  }

  .limit-card-divider {
    height: 1px;
    background: #E5E7EB;
  }

  .limit-info-section {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 16px;
  }

  .slider-section {
    display: flex;
    flex-direction: column;
    gap: 12px;
    padding: 16px 20px 20px 20px;
  }

  .slider-value-display {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .slider-value-amount {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 700;
    font-size: 28px;
    line-height: 1.2;
    color: #111827;
    font-variant-numeric: tabular-nums;
  }

  /* CRED-style slider */
  .shadcn-slider {
    position: relative;
    display: flex;
    width: 100%;
    touch-action: none;
    user-select: none;
    height: 28px;
    align-items: center;
  }

  .slider-track {
    position: relative;
    height: 8px;
    width: 100%;
    overflow: hidden;
    border-radius: 9999px;
    background: linear-gradient(90deg, #E5E7EB 0%, #D1D5DB 100%);
    box-shadow: inset 0 1px 2px rgba(0,0,0,0.1);
  }

  .slider-range {
    position: absolute;
    height: 100%;
    background: linear-gradient(90deg, #22C55E 0%, #15803D 100%);
    border-radius: 9999px;
    box-shadow: 0 0 12px rgba(21, 128, 61, 0.4);
  }

  .slider-input {
    position: absolute;
    width: 100%;
    height: 28px;
    opacity: 0;
    cursor: pointer;
    z-index: 3;
    margin: 0;
  }

  .slider-thumb {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    height: 24px;
    width: 24px;
    border-radius: 9999px;
    background: linear-gradient(145deg, #FFFFFF 0%, #F3F4F6 100%);
    border: 2px solid #15803D;
    box-shadow: 
      0 2px 8px rgba(21, 128, 61, 0.3),
      0 0 0 0 rgba(21, 128, 61, 0.2),
      inset 0 1px 0 rgba(255,255,255,0.8);
    pointer-events: none;
    z-index: 2;
    transition: box-shadow 0.2s ease, transform 0.2s ease;
  }

  .slider-input:active ~ .slider-thumb {
    box-shadow: 
      0 4px 16px rgba(21, 128, 61, 0.4),
      0 0 0 6px rgba(21, 128, 61, 0.15),
      inset 0 1px 0 rgba(255,255,255,0.8);
    transform: translateY(-50%) scale(1.1);
  }

  .slider-labels {
    display: flex;
    justify-content: space-between;
    padding: 0 2px;
  }

  .slider-min, .slider-max {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 500;
    font-size: 12px;
    color: #9CA3AF;
  }

  .limit-info-card {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 16px;
    background: #F0FDF4;
    border-radius: 12px;
    border: 1px solid #BBF7D0;
  }

  .cash-stack-icon {
    width: 48px;
    height: 48px;
    flex-shrink: 0;
  }

  .limit-info-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 500;
    font-size: 13px;
    line-height: 1.5;
    color: #15803D;
    flex: 1;
  }

  .edit-limit-footer {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  .btn-text-link {
    background: none;
    border: none;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    color: #184595;
    cursor: pointer;
    padding: 12px 8px;
    transition: opacity 0.15s;
  }
  .btn-text-link:active { opacity: 0.7; }
</style>
