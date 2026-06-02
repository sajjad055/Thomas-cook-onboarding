<script lang="ts">
  import { goto } from '$app/navigation';
  import { onMount } from 'svelte';
  import StatusBar from '$lib/components/StatusBar.svelte';
  import ProgressComponent from '$lib/components/ProgressComponent.svelte';

  let aadhaar = $state('');
  let agreed = $state(false);
  let loading = $state(false);
  let touched = $state(false);
  let inputEl = $state<HTMLInputElement | null>(null);

  onMount(() => {
    setTimeout(() => inputEl?.focus(), 400);
  });

  // Aadhaar: 12 digits
  let rawDigits = $derived(aadhaar.replace(/\D/g, '').slice(0, 12));
  let isValid = $derived(rawDigits.length === 12);
  let showError = $derived(touched && rawDigits.length > 0 && !isValid);

  // Format as XXXX XXXX XXXX for display
  let displayValue = $derived(
    rawDigits.replace(/(\d{4})(?=\d)/g, '$1 ')
  );

  function onInput(e: Event) {
    const input = e.target as HTMLInputElement;
    // Strip non-digits, cap at 12
    const digits = input.value.replace(/\D/g, '').slice(0, 12);
    aadhaar = digits;
    // Set formatted value back (with spaces)
    // Use requestAnimationFrame to avoid cursor jump
    requestAnimationFrame(() => {
      input.value = digits.replace(/(\d{4})(?=\d)/g, '$1 ');
    });
  }

  async function handleVerify() {
    touched = true;
    if (!isValid || !agreed) return;
    loading = true;
    await new Promise(r => setTimeout(r, 600));
    loading = false;
    goto('/pan');
  }
</script>

<svelte:head><title>Enter Aadhaar – DCB</title></svelte:head>

<div class="screen">

  <!-- ── HEADER ── -->
  <div class="header-area">
    <StatusBar />

    <div class="icon-row">
      <button class="icon-btn" aria-label="Back" onclick={() => goto('/mobile/otp')}>
        <i class="ph ph-caret-left"></i>
      </button>
      <div class="icon-group">
        <button class="icon-btn" aria-label="Help">
          <i class="ph-duotone ph-headset" style="font-size:24px"></i>
        </button>
      </div>
    </div>

    <div class="progress-row">
      <h1 class="screen-title">Enter Aadhaar number</h1>
      <ProgressComponent step={2} totalSteps={9} />
    </div>
  </div>

  <!-- ── FORM AREA ── -->
  <div class="form-area">
    <div class="form-inner">

      <!-- KYC Card: SVG background + input overlay -->
      <div class="kyc-card-wrap" class:aadhaar-complete={isValid}>
        <!-- Tricolor border overlay (visible when complete) -->
        <div class="tricolor-border" class:active={isValid}></div>
        <img
          src="/aadhaar-front.svg"
          alt="Aadhaar card"
          class="kyc-bg"
          draggable="false"
        />

        <!-- Input overlaid on the card, positioned where the placeholder text sits -->
        <div class="kyc-input-overlay">
          <input
            bind:this={inputEl}
            type="tel"
            inputmode="numeric"
            placeholder="Enter Aadhaar number"
            value={displayValue}
            oninput={onInput}
            onblur={() => touched = true}
            class="aadhaar-input"
            class:has-value={rawDigits.length > 0}
            maxlength="14"
            autocomplete="off"
          />
        </div>
      </div>

      {#if showError}
        <p class="error-text">Please enter a valid 12-digit Aadhaar number</p>
      {/if}

    </div>
  </div>

  <div class="spacer"></div>

  <!-- ── FOOTER ── -->
  <div class="footer">
    <!-- Consent checkbox -->
    <div class="consent-row">
      <button class="checkbox" class:checked={agreed} onclick={() => agreed = !agreed} aria-label="Agree to terms">
        {#if agreed}
          <div class="check-anim">
            <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
              <rect width="18" height="18" rx="4" fill="#242A80"/>
              <path d="M5.5 9.2L7.8 11.6L12.5 6.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </div>
        {:else}
          <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
            <rect x="0.5" y="0.5" width="17" height="17" rx="3.5" stroke="#D1D5DB"/>
          </svg>
        {/if}
      </button>
      <div class="consent-text-wrap">
        <p class="consent-body">
          I agree and authorize DCB Bank Limited to fetch my personal details from UIDAI. I hereby......
          <button class="read-more" onclick={() => {}}>Read more</button>
        </p>
      </div>
    </div>

    <!-- CTA -->
    <div class="cta-wrap">
      <button
        class="btn-primary"
        disabled={!isValid || !agreed || loading}
        onclick={handleVerify}
      >
        {#if loading}
          <span class="spinner"></span>
        {:else}
          Verify Aadhaar
        {/if}
      </button>
    </div>
  </div>

</div>

<style>
  .screen {
    width: 100%;
    min-height: 100dvh;
    display: flex;
    flex-direction: column;
    background: #FFFDF6;
  }

  /* ── Header ── */
  .header-area {
    background: #FFFDF6;
    padding-bottom: 16px;
    flex-shrink: 0;
    display: flex;
    flex-direction: column;
    gap: 24px;
  }

  .icon-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
  }
  .icon-group { display: flex; align-items: center; }

  .icon-btn {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 48px;
    height: 48px;
    background: none;
    border: none;
    cursor: pointer;
    border-radius: 50%;
    transition: background 0.15s;
  }
  .icon-btn:active { background: rgba(0,0,0,0.06); }

  .progress-row {
    display: flex;
    align-items: flex-end;
    justify-content: space-between;
    padding: 0 16px;
    gap: 10px;
  }

  .screen-title {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 24px;
    line-height: 1.2;
    color: #111827;
    letter-spacing: -0.25px;
    flex: 1;
  }

  /* ── Form ── */
  .form-area {
    padding: 20px 0 0;
    border-radius: 16px 16px 0 0;
    flex-shrink: 0;
  }

  .form-inner {
    display: flex;
    flex-direction: column;
    gap: 8px;
    padding: 0 16px;
  }

  /* ── KYC Card with SVG bg ── */
  .kyc-card-wrap {
    position: relative;
    width: 100%;
    border-radius: 16px;
    overflow: visible;
    transition: box-shadow 0.5s ease;
  }

  /* Tricolor border — animated rotating gradient ring */
  .tricolor-border {
    position: absolute;
    inset: -1px;
    border-radius: 17px;
    pointer-events: none;
    z-index: 0;
    opacity: 0;
    transition: opacity 0.5s ease;
    overflow: hidden;
  }
  .tricolor-border::before {
    content: '';
    position: absolute;
    inset: -50%;
    background: conic-gradient(
      #FF9933,
      #FFB366,
      #FFFFFF,
      #FFFFFF,
      #66BB6A,
      #138808,
      #138808,
      #66BB6A,
      #FFFFFF,
      #FFFFFF,
      #FFB366,
      #FF9933
    );
    animation: tricolorSpin 8s linear infinite;
  }
  .tricolor-border::after {
    content: '';
    position: absolute;
    inset: 1px;
    border-radius: 16px;
    background: transparent;
    z-index: 1;
  }
  .tricolor-border.active {
    opacity: 1;
  }

  @keyframes tricolorSpin {
    0%   { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }

  /* No shadow when complete */
  .kyc-card-wrap.aadhaar-complete {
  }

  .kyc-bg {
    display: block;
    width: 100%;
    height: auto;
    object-fit: contain;
    user-select: none;
    pointer-events: none;
    position: relative;
    z-index: 1;
    border-radius: 16px;
  }

  /* Input overlay — positioned over the card where the Aadhaar number goes
     Based on Figma: the input sits in the lower-center of the card,
     roughly at y:120 from top, spanning most of the card width */
  .kyc-input-overlay {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    display: flex;
    align-items: center;
    justify-content: center;
    width: 260px;
    z-index: 2;
  }

  .aadhaar-input {
    width: 100%;
    background: none;
    border: none;
    outline: none;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 20px;
    line-height: 1.2;
    color: #212121;
    letter-spacing: 2px;
    caret-color: #242A80;
    text-align: center;
  }
  .aadhaar-input::placeholder {
    color: rgba(33,33,33,0.35);
    font-weight: 600;
    font-size: 16px;
    letter-spacing: 0;
    text-align: center;
  }

  .error-text {
    font-family: 'Nunito Sans', sans-serif;
    font-size: 12px;
    color: #B91C1C;
    padding: 0 4px;
  }

  /* ── Spacer ── */
  .spacer { flex: 1; }

  /* ── Footer ── */
  .footer { position: sticky; bottom: 0; z-index: 5; margin-top: auto;
    display: flex;
    flex-direction: column;
    gap: 16px;
    padding: 20px 0 16px;
    flex-shrink: 0;
  }

  /* Consent */
  .consent-row {
    display: flex;
    align-items: flex-start;
    gap: 8px;
    padding: 0 16px;
  }

  .checkbox {
    width: 24px;
    height: 24px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 3px;
    flex-shrink: 0;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .check-anim {
    animation: checkPop 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
    display: flex;
  }
  @keyframes checkPop {
    0%   { transform: scale(0); opacity: 0; }
    60%  { transform: scale(1.15); opacity: 1; }
    100% { transform: scale(1); }
  }

  .consent-text-wrap {
    flex: 1;
  }

  .consent-body {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 1.5;
    color: #6B7280;
  }

  .read-more {
    background: none;
    border: none;
    cursor: pointer;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 12px;
    line-height: 1.5;
    color: #111827;
    padding: 0;
    display: inline;
  }

  /* CTA */
  .cta-wrap {
    padding: 0 16px;
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
    box-shadow: 0px 4px 0px #06142A;
    transition: opacity 0.15s, transform 0.1s;
  }
  .btn-primary:active:not(:disabled) {
    opacity: 0.88;
    transform: scale(0.99);
  }
  .btn-primary:disabled {
    background: #D1D5DB;
    box-shadow: none;
    cursor: not-allowed;
  }

  .spinner {
    width: 20px;
    height: 20px;
    border: 2.5px solid rgba(255,255,255,0.4);
    border-top-color: #FFFFFF;
    border-radius: 50%;
    animation: spin 0.7s linear infinite;
  }
  @keyframes spin { to { transform: rotate(360deg); } }
</style>
