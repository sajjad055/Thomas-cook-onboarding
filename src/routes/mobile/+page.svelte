<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import { fade } from 'svelte/transition';
  import { onMount } from 'svelte';
  import StatusBar from '$lib/components/StatusBar.svelte';

  let mobile = $state('');
  let touched = $state(false);
  let loading = $state(false);
  let focused = $state(false);

  onMount(() => {
    setTimeout(() => {
      const el = document.getElementById('mobile-input');
      el?.focus();
      focused = true;
    }, 400);
  });

  // Validation
  let isValid = $derived(/^[6-9]\d{9}$/.test(mobile));
  let showError = $derived(touched && mobile.length > 0 && !isValid);
  let showLengthError = $derived(touched && mobile.length === 0);

  function onInput(e: Event) {
    const val = (e.target as HTMLInputElement).value.replace(/\D/g, '').slice(0, 10);
    mobile = val;
  }

  function onBlur() {
    touched = true;
    focused = false;
  }

  async function handleGetOTP() {
    touched = true;
    if (!isValid) return;
    loading = true;
    await new Promise(r => setTimeout(r, 600));
    loading = false;
    goto(`${base}/mobile/otp`);
  }
</script>

<svelte:head><title>Your mobile number – IOB</title></svelte:head>

<div class="screen">

  <!-- ── HEADER AREA (grey bg) ── -->
  <div class="header-area">
    <StatusBar />

    <!-- Back + Help icons row -->
    <div class="icon-row">
    <!-- Back | headset_mic + close -->
    <div class="icon-row">
      <button class="icon-btn" aria-label="Back" onclick={() => goto(`${base}/existing-customer`)}>
        <i class="ph ph-caret-left" style="font-size:24px; color:#111827"></i>
      </button>
      <div class="icon-group">
        <button class="icon-btn" aria-label="Help">
          <i class="ph-duotone ph-headset" style="font-size:24px; color:#111827"></i>
        </button>
      </div>
    </div>
    </div>

    <!-- Illustration frame -->
    <div class="illustration-frame">
      <img src="{base}/Phone.svg" alt="Phone" class="illustration-img" />
    </div>

    <!-- Progress row: title + ring -->
    <div class="progress-row">
      <h1 class="screen-title">Verify your mobile number</h1>
    </div>
  </div>

  <!-- ── FORM AREA (white, rounded top) ── -->
  <div class="form-area">
    <div class="form-inner">

      <!-- Phone field matching TextField structure -->
      <div class="field-group">
        <!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
        <div
          class="tf"
          class:active={focused || mobile.length > 0}
          class:focused
          class:error={showError}
          onclick={() => { const el = document.getElementById('mobile-input'); el?.focus(); }}
        >
          <div class="tf-content" class:active={focused || mobile.length > 0}>
            <span class="tf-label" class:floated={focused || mobile.length > 0}>Enter Mobile number</span>
            <div class="phone-row" class:shown={focused || mobile.length > 0}>
              <span class="country-code">+91</span>
              <input
                id="mobile-input"
                type="tel"
                inputmode="numeric"
                autocomplete="tel"
                placeholder={focused ? '9XXXXXXXXX' : ''}
                value={mobile}
                oninput={onInput}
                onblur={onBlur}
                onfocus={() => focused = true}
                class="phone-input"
                maxlength="10"
              />
            </div>
          </div>
        </div>

        {#if showError}
          <p class="tf-helper tf-error" in:fade={{ duration: 150 }}>
            Please enter a valid 10-digit mobile number
          </p>
        {:else if showLengthError}
          <p class="tf-helper tf-error" in:fade={{ duration: 150 }}>
            Mobile number is required
          </p>
        {/if}
      </div>

    </div>
  </div>

  <!-- ── SPACER ── -->
  <div class="spacer"></div>

  <!-- Bank Partner Logos -->
  <div class="partner-logos">
    <img src="{base}/logos-bank-partner.svg" alt="Bank Partners" class="partner-img" draggable="false" />
  </div>

  <!-- ── FOOTER BUTTON ── -->
  <div class="footer">
    <button
      class="btn-primary"
      class:loading
      disabled={loading}
      onclick={handleGetOTP}
    >
      {#if loading}
        <span class="spinner"></span>
      {:else}
        Get OTP
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

  /* ── Header ── */
  .header-area {
    background: #FFFCF4;
    padding-bottom: 0;
    flex-shrink: 0;
    display: flex;
    flex-direction: column;
    gap: 24px;
  }

  .icon-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0;
    width: 100%;
  }
  .icon-group {
    display: flex;
    align-items: center;
    gap: 0;
  }

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

  .illustration-frame {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    background: #F3F4F6;
    margin-left: 16px;
    flex-shrink: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
  }

  .illustration-img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .progress-row {
    display: flex;
    align-items: flex-end;
    justify-content: space-between;
    padding: 16px 16px 0 16px;
    gap: 16px;
  }

  .screen-title {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 24px;
    line-height: 1.2;
    color: #212121;
    letter-spacing: -0.25px;
    flex: 1;
  }

  /* ── Form ── */
  .form-area {
    background: #FFFCF4;
    border-radius: 16px 16px 0 0;
    padding: 24px 0 0;
    flex-shrink: 0;
  }

  .form-inner {
    display: flex;
    flex-direction: column;
    gap: 16px;
    padding: 0 16px;
  }

  .field-group {
    display: flex;
    flex-direction: column;
    gap: 6px;
  }

  /* Match TextField component styles */
  .tf {
    display: flex;
    align-items: center;
    gap: 16px;
    background: #FFFFFF;
    border: 1px solid #D1D5DB;
    border-radius: 8px;
    padding: 10px 12px;
    min-height: 52px;
    cursor: text;
    transition: border-color 0.15s ease;
  }
  .tf.focused { border-color: #D1D5DB; }
  .tf.error { border-color: #B91C1C; }

  .tf-content {
    display: flex;
    flex-direction: column;
    flex: 1;
    min-width: 0;
    justify-content: center;
  }
  .tf-content.active {
    gap: 4px;
  }

  .tf-label {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 1.5;
    color: #6B7280;
    pointer-events: none;
    transition: color 0.18s ease;
  }
  .tf-label.floated {
    color: #111827;
  }

  .phone-row {
    display: flex;
    align-items: center;
    gap: 8px;
    height: 0;
    opacity: 0;
    overflow: hidden;
    transition: height 0.15s ease, opacity 0.15s ease;
  }
  .phone-row.shown {
    height: 20px;
    opacity: 1;
  }

  .country-code {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    color: #111827;
    flex-shrink: 0;
  }

  .phone-input {
    flex: 1;
    background: none;
    border: none;
    outline: none;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    color: #111827;
    padding: 0;
    line-height: 1.43;
  }
  .phone-input::placeholder {
    color: #9CA3AF;
    font-weight: 400;
  }

  .tf-helper {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 1.5;
    color: #6B7280;
    padding: 0 4px;
  }
  .tf-error { color: #B91C1C; }

  /* ── Spacer ── */
  .spacer { flex: 1; }

  /* ── Partner Logos ── */
  .partner-logos {
    display: flex;
    justify-content: center;
    padding: 40px 16px 0;
  }
  .partner-img {
    height: 46px;
    filter: grayscale(100%);
    width: auto;
  }

  /* ── Footer ── */
  .footer { position: sticky; bottom: 0; z-index: 5; margin-top: auto;
    padding: 16px 16px calc(40px + env(safe-area-inset-bottom));
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
    box-shadow: 0px 4px 0px #06142A;
    transition: opacity 0.15s ease, transform 0.1s ease;
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

  /* Spinner */
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
