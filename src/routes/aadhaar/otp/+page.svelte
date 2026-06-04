<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import { fade } from 'svelte/transition';
  import StatusBar from '$lib/components/StatusBar.svelte';
  import { onMount, onDestroy } from 'svelte';

  // 6 OTP boxes
  let otp = $state(['', '', '', '', '', '']);
  let inputs: HTMLInputElement[] = $state([]);
  let loading = $state(false);
  let error = $state('');

  // Resend timer — 30s
  let resendSeconds = $state(30);
  let canResend = $state(false);
  let timerInterval: ReturnType<typeof setInterval>;

  function startTimer() {
    resendSeconds = 30;
    canResend = false;
    clearInterval(timerInterval);
    timerInterval = setInterval(() => {
      resendSeconds--;
      if (resendSeconds <= 0) {
        clearInterval(timerInterval);
        canResend = true;
      }
    }, 1000);
  }

  onMount(() => {
    startTimer();
    inputs[0]?.focus();
  });
  onDestroy(() => clearInterval(timerInterval));

  let isComplete = $derived(otp.every(d => d !== ''));

  function handleInput(i: number, e: Event) {
    const val = (e.target as HTMLInputElement).value.replace(/\D/g, '');
    if (!val) return;
    otp[i] = val.slice(-1);
    error = '';
    if (i < 5) inputs[i + 1]?.focus();
  }

  function handleKeydown(i: number, e: KeyboardEvent) {
    if (e.key === 'Backspace') {
      if (otp[i]) {
        otp[i] = '';
      } else if (i > 0) {
        otp[i - 1] = '';
        inputs[i - 1]?.focus();
      }
    }
    if (e.key === 'ArrowLeft' && i > 0) inputs[i - 1]?.focus();
    if (e.key === 'ArrowRight' && i < 5) inputs[i + 1]?.focus();
  }

  function handlePaste(e: ClipboardEvent) {
    e.preventDefault();
    const text = e.clipboardData?.getData('text').replace(/\D/g, '').slice(0, 6) ?? '';
    text.split('').forEach((ch, i) => { otp[i] = ch; });
    inputs[Math.min(text.length, 5)]?.focus();
  }

  async function handleContinue() {
    if (!isComplete) return;
    loading = true;
    error = '';
    await new Promise(r => setTimeout(r, 600));
    loading = false;
    goto(`${base}/pan`);
  }

  function handleResend() {
    if (!canResend) return;
    otp = ['', '', '', '', '', ''];
    error = '';
    startTimer();
    inputs[0]?.focus();
  }
</script>

<svelte:head><title>Aadhaar OTP – IOB</title></svelte:head>

<div class="screen">

  <!-- ── HEADER — white bg ── -->
  <div class="header-area">
    <StatusBar />

    <!-- Back | headset_mic + close -->
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
  </div>

  <!-- ── CONTENT ── -->
  <div class="content">

    <!-- Title block  gap:4px  px:16px -->
    <div class="title-block">
      <h1 class="title">Enter OTP</h1>
      <p class="subtitle">Enter OTP sent to your Aadhaar-linked mobile number</p>
    </div>

    <!-- OTP + resend  gap:32px -->
    <div class="otp-section">

      <!-- OTP row  gap:8px  px:16px -->
      <div class="otp-row" onpaste={handlePaste}>
        {#each otp as digit, i}
          <div class="otp-box" class:filled={!!digit} class:has-error={!!error}>
            <input
              bind:this={inputs[i]}
              type="tel"
              inputmode="numeric"
              maxlength="1"
              value={digit}
              oninput={(e) => handleInput(i, e)}
              onkeydown={(e) => handleKeydown(i, e)}
              class="otp-input"
              aria-label="OTP digit {i + 1}"
            />
          </div>
        {/each}
      </div>

      <!-- Resend row  centered -->
      <div class="resend-row">
        {#if canResend}
          <button class="resend-btn" onclick={handleResend}>
            Not received? <span class="resend-link">Resend OTP</span>
          </button>
        {:else}
          <p class="resend-text">
            Not received? Resend OTP
          </p>
        {/if}
      </div>

    </div>

    {#if error}
      <p class="error-text" in:fade={{ duration: 150 }}>{error}</p>
    {/if}

  </div>

  <div class="spacer"></div>

  <!-- ── FOOTER  pt:20px pb:20px ── -->
  <div class="footer">
    <button
      class="btn-primary"
      disabled={!isComplete || loading}
      onclick={handleContinue}
    >
      {#if loading}
        <span class="spinner"></span>
      {:else}
        Continue
      {/if}
    </button>
  </div>

</div>

<style>
  /* ── Screen ── */
  .screen {
    width: 100%;
    min-height: 100dvh;
    display: flex;
    flex-direction: column;
    background: #FFFCF4;
  }

  /* ── Header — white bg, gap:24px ── */
  .header-area {
    background: #FFFCF4;
    flex-shrink: 0;
    display: flex;
    flex-direction: column;
    gap: 24px;
    padding-bottom: 24px;
  }

  .icon-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
  }

  .icon-group {
    display: flex;
    align-items: center;
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

  /* ── Content  gap:24px  px:16px ── */
  .content {
    display: flex;
    flex-direction: column;
    gap: 24px;
    padding: 0 0 0 0;
    flex-shrink: 0;
  }

  /* Title block  gap:4px  px:16px */
  .title-block {
    display: flex;
    flex-direction: column;
    gap: 4px;
    padding: 0 16px;
  }

  .title {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 20px;
    line-height: 1.2;
    color: #111827;
    letter-spacing: -0.25px;
  }

  .subtitle {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 1.5;
    color: #6B7280;
  }

  .subtitle-bold {
    font-weight: 600;
    color: #6B7280;
  }

  /* OTP section  gap:32px  centered */
  .otp-section {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 32px;
  }

  /* OTP row  gap:8px  px:16px */
  .otp-row {
    display: flex;
    gap: 8px;
    padding: 0 16px;
    width: 100%;
  }

  /* OTP box — fill width equally, h:48px, no fill, border:#D1D5DB, radius:8px */
  .otp-box {
    flex: 1;
    height: 52px;
    background: #FFFFFF;
    border: 1px solid #D1D5DB;
    border-radius: 8px;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: border-color 0.15s;
  }
  .otp-box.filled {
    background: #FFFFFF;
    border-color: #D1D5DB;
  }
  .otp-box.has-error { border-color: #B91C1C; }

  .otp-input {
    width: 100%;
    height: 100%;
    background: none;
    border: none;
    outline: none;
    text-align: center;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    color: #000000;
    caret-color: #6F43C0;
  }

  /* Resend row — centered, Poppins 400 14px */
  .resend-row {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
  }

  .resend-text {
    font-family: 'Poppins', 'Manrope', sans-serif;
    font-weight: 400;
    font-size: 14px;
    line-height: 1.5;
    color: #6B7280;
    text-align: center;
  }

  .resend-btn {
    background: none;
    border: none;
    cursor: pointer;
    font-family: 'Poppins', 'Manrope', sans-serif;
    font-weight: 400;
    font-size: 14px;
    color: #6B7280;
    text-align: center;
  }

  .resend-link {
    color: #184595;
    font-weight: 600;
    text-decoration: underline;
  }

  .error-text {
    font-family: 'Nunito Sans', sans-serif;
    font-size: 12px;
    color: #B91C1C;
    text-align: center;
    padding: 0 16px;
  }

  /* ── Spacer ── */
  .spacer { flex: 1; }

  /* ── Footer  pt:20px pb:20px  px:16px ── */
  .footer { position: sticky; bottom: 0; z-index: 5; margin-top: auto;
    padding: 20px 16px calc(40px + env(safe-area-inset-bottom));
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
