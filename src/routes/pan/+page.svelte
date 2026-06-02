<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import StatusBar from '$lib/components/StatusBar.svelte';
  import TextField from '$lib/components/TextField.svelte';
  import ToastMessage from '$lib/components/ToastMessage.svelte';

  let pan = $state('');
  let dob = $state('17/08/1999');
  let name = $state('Sajjad S');
  let agreed = $state(false);
  let loading = $state(false);
  let panTouched = $state(false);
  let showToast = $state(false);

  onMount(() => {
    showToast = true;
    setTimeout(() => { showToast = false; }, 2000);
  });

  // PAN: 5 letters, 4 digits, 1 letter
  let panUpper = $derived(pan.toUpperCase());
  let isPanValid = $derived(/^[A-Z]{5}\d{4}[A-Z]$/.test(panUpper));
  let panError = $derived(panTouched && pan.length > 0 && !isPanValid ? 'Please enter a valid PAN number' : '');

  function onPanInput(e: Event) {
    pan = (e.target as HTMLInputElement).value.toUpperCase().replace(/[^A-Z0-9]/g, '').slice(0, 10);
  }

  function onDobInput(e: Event) {
    let raw = (e.target as HTMLInputElement).value.replace(/\D/g, '').slice(0, 8);
    if (raw.length > 4) raw = raw.slice(0, 2) + '/' + raw.slice(2, 4) + '/' + raw.slice(4);
    else if (raw.length > 2) raw = raw.slice(0, 2) + '/' + raw.slice(2);
    dob = raw;
  }

  let isDobValid = $derived(/^\d{2}\/\d{2}\/\d{4}$/.test(dob));
  let allValid = $derived(isPanValid && isDobValid && name.trim().length > 0);

  async function handleVerify() {
    panTouched = true;
    if (!allValid || !agreed) return;
    loading = true;
    await new Promise(r => setTimeout(r, 600));
    loading = false;
    goto(`${base}/income`);
  }
</script>

<svelte:head><title>Enter PAN details – IOB</title></svelte:head>

<div class="screen">

  <!-- ── HEADER ── -->
  <div class="header-area">
    <StatusBar />
    <div class="icon-row">
      <button class="icon-btn" aria-label="Back" onclick={() => goto(`${base}/aadhaar`)}>
        <i class="ph ph-caret-left" style="font-size:24px; color:#111827"></i>
      </button>
      <div class="icon-group">
        <button class="icon-btn" aria-label="Help">
          <i class="ph-duotone ph-headset" style="font-size:24px; color:#111827"></i>
        </button>
      </div>
    </div>
    <!-- Illustration frame -->
    <div class="illustration-frame">
      <img src="{base}/security.svg" alt="Security" class="illustration-img" />
    </div>

    <div class="progress-row">
      <h1 class="screen-title">Enter PAN details</h1>
    </div>
  </div>

  <!-- ── FORM ── -->
  <div class="form-area">
    <div class="form-scroll">

      <TextField
        label="PAN number"
        bind:value={panUpper}
        placeholder="ABCDE1234F"
        maxlength={10}
        autocapitalize="characters"
        autofocus={true}
        oninput={onPanInput}
        onblur={() => panTouched = true}
        errorText={panError}
        supportText='Check a PAN sample for clarity. <button onclick="">View Sample</button>'
      />

      <TextField
        label="Date of birth"
        bind:value={dob}
        type="tel"
        inputmode="numeric"
        placeholder="DD/MM/YYYY"
        maxlength={10}
        oninput={onDobInput}
        endIcon="calendar_month"
      />

      <TextField
        label="Name as per PAN"
        bind:value={name}
        placeholder="Enter name"
        supportText="This is pre-filled from Aadhaar. Please enter the exact name in PAN card to avoid verification issues"
      />

    </div>
  </div>

  <div class="spacer"></div>

  <!-- ── FOOTER ── -->
  <div class="footer">
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
          I hereby authorize IOB Bank (India) Ltd. ("Bank") to fetch/retrieve my Know Your Customer..
          <button class="read-more" onclick={() => {}}>Read more</button>
        </p>
      </div>
    </div>
    <div class="cta-wrap">
      <button class="btn-primary" disabled={!allValid || !agreed || loading} onclick={handleVerify}>
        {#if loading}<span class="spinner"></span>{:else}Verify PAN details{/if}
      </button>
    </div>
  </div>

  <!-- Toast -->
  <div class="toast-position">
    <ToastMessage state="success" message="Aadhaar successfully verified" visible={showToast} onclose={() => { showToast = false; }} />
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

  .toast-position {
    position: fixed;
    bottom: 90px;
    left: 16px;
    right: 16px;
    z-index: 100;
  }

  .header-area {
    background: #FFFCF4;
    padding-bottom: 0;
    flex-shrink: 0;
    display: flex;
    flex-direction: column;
    gap: 24px;
  }

  .icon-row { display: flex; align-items: center; justify-content: space-between; width: 100%; }
  .icon-group { display: flex; align-items: center; }
  .icon-btn {
    display: flex; align-items: center; justify-content: center;
    width: 48px; height: 48px; background: none; border: none;
    cursor: pointer; border-radius: 50%; transition: background 0.15s;
  }
  .icon-btn:active { background: rgba(0,0,0,0.06); }

  .illustration-frame {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    background: #F3F4F6;
    border: 0.5px solid #F5F5F5;
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

  .progress-row { display: flex; align-items: flex-end; justify-content: space-between; padding: 16px 16px 0 16px; gap: 16px; }
  .title-with-illustration { display: flex; align-items: flex-end; gap: 8px; flex: 1; }
  .pan-illustration { flex-shrink: 0; user-select: none; pointer-events: none; }
  .screen-title {
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 24px;
    line-height: 1.2; color: #111827; letter-spacing: -0.25px;
  }

  .form-area { padding: 24px 0 0; border-radius: 16px 16px 0 0; flex-shrink: 0; }
  .form-scroll { display: flex; flex-direction: column; gap: 16px; padding: 0 16px; }

  .spacer { flex: 1; }

  .footer { position: sticky; bottom: 0; z-index: 5; margin-top: auto; display: flex; flex-direction: column; gap: 16px; padding: 20px 0 16px; flex-shrink: 0; }
  .consent-row { display: flex; align-items: flex-start; gap: 8px; padding: 0 16px; }
  .checkbox {
    width: 24px; height: 24px; background: none; border: none;
    cursor: pointer; padding: 3px; flex-shrink: 0;
    display: flex; align-items: center; justify-content: center;
  }
  .check-anim { animation: checkPop 0.3s cubic-bezier(0.34, 1.56, 0.64, 1); display: flex; }
  @keyframes checkPop { 0% { transform: scale(0); opacity: 0; } 60% { transform: scale(1.15); opacity: 1; } 100% { transform: scale(1); } }
  .consent-text-wrap { flex: 1; }
  .consent-body { font-family: 'Nunito Sans', sans-serif; font-weight: 400; font-size: 12px; line-height: 1.5; color: #6B7280; }
  .read-more { background: none; border: none; cursor: pointer; font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 12px; line-height: 1.5; color: #111827; padding: 0; display: inline; }
  .cta-wrap { padding: 0 16px; }
  .btn-primary {
    display: flex; align-items: center; justify-content: center;
    width: 100%; height: 48px; background: #184595; color: #FFFFFF;
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 16px;
    border: none; border-radius: 8px; cursor: pointer;
    box-shadow: 0px 4px 0px #06142A; transition: opacity 0.15s, transform 0.1s;
  }
  .btn-primary:active:not(:disabled) { opacity: 0.88; transform: scale(0.99); }
  .btn-primary:disabled { background: #D1D5DB; box-shadow: none; cursor: not-allowed; }
  .spinner { width: 20px; height: 20px; border: 2.5px solid rgba(255,255,255,0.4); border-top-color: #FFFFFF; border-radius: 50%; animation: spin 0.7s linear infinite; }
  @keyframes spin { to { transform: rotate(360deg); } }
</style>
