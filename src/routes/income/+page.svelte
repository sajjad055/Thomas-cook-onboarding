<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import { onMount, onDestroy } from 'svelte';
  import { fade, scale } from 'svelte/transition';
  import { cubicOut } from 'svelte/easing';
  import { browser } from '$app/environment';
  import StatusBar from '$lib/components/StatusBar.svelte';
  import ProgressiveInputStepper from '$lib/components/ProgressiveInputStepper.svelte';
  import ToastMessage from '$lib/components/ToastMessage.svelte';
  import BottomSheet from '$lib/components/BottomSheet.svelte';

  let lottieReady = $state(false);

  let showToast = $state(false);

  onMount(() => {
    showToast = true;
    setTimeout(() => { showToast = false; }, 2000);
    if (browser) {
      import('@lottiefiles/dotlottie-wc').catch(() => {});
    }
    lottieReady = true;
  });

  // Stepper steps
  let steps = $state([
    {
      label: 'Your occupation',
      type: 'chips',
      options: ['Salaried', 'Self-employed', 'Business owner', 'Freelancer / Consultant', 'Retired', 'Student', 'Homemaker'],
      value: ''
    },
    {
      label: 'Enter your company name',
      type: 'text',
      placeholder: 'e.g. Acme Corp',
      value: ''
    },
    {
      label: 'Gross annual income',
      type: 'chips',
      options: ['Below ₹2.5 lakhs', '₹2.5 to 5 lakhs', '₹5 to 10 lakhs', '₹10 to 25 lakhs', '₹25 to 50 lakhs', 'Above ₹50 lakhs'],
      value: ''
    },
    {
      label: 'Primary source of income',
      type: 'chips',
      options: ['Salary', 'Business income', 'Rental income', 'Investment returns', 'Pension', 'Agriculture', 'Other'],
      value: ''
    },
    {
      label: 'Your net worth',
      type: 'chips',
      options: ['₹2 to 5 lakhs', '₹5 to 10 lakhs', '₹10 to 25 lakhs', '₹25 to 50 lakhs', '₹50 lakhs to 1 crore', 'Above ₹1 crore'],
      value: ''
    }
  ]);

  let loading = $state(false);
  let aaAgreed = $state(false);
  let allValid = $derived(steps.every(s => s.value !== ''));

  function handleStepChange(stepIndex: number, value: string) {
    steps[stepIndex].value = value;
  }

  // OTP bottom sheet
  let showOtpSheet = $state(false);
  let otp = $state(['', '', '', '', '', '']);
  let otpInputs: HTMLInputElement[] = $state([]);
  let otpError = $state('');
  let otpLoading = $state(false);
  let resendSeconds = $state(30);
  let canResend = $state(false);
  let timerInterval: ReturnType<typeof setInterval>;

  // Verification method choice sheet
  let showMethodSheet = $state(false);
  let selectedMethod = $state<'aggregator' | 'upload' | null>(null);

  // Upload sheet
  let showUploadSheet = $state(false);
  let uploadedFiles = $state<File[]>([]);
  let uploadLoading = $state(false);
  let fileInput: HTMLInputElement;

  function openMethodSheet() {
    if (!allValid) return;
    selectedMethod = 'aggregator'; // Default to AA
    aaAgreed = false;
    showMethodSheet = true;
  }

  function handleMethodContinue() {
    if (!selectedMethod) return;
    
    if (selectedMethod === 'aggregator') {
      if (!aaAgreed) return; // Need consent for AA
      showMethodSheet = false;
      // Proceed with OTP flow
      otp = ['', '', '', '', '', ''];
      otpError = '';
      showOtpSheet = true;
      startTimer();
      setTimeout(() => otpInputs[0]?.focus(), 300);
    } else {
      // Open upload sheet (no consent needed)
      showMethodSheet = false;
      uploadedFiles = [];
      showUploadSheet = true;
    }
  }

  function handleFileSelect(e: Event) {
    const input = e.target as HTMLInputElement;
    if (input.files) {
      const newFiles = Array.from(input.files);
      uploadedFiles = [...uploadedFiles, ...newFiles].slice(0, 3); // Max 3 files
    }
    input.value = ''; // Reset so same file can be selected again
  }

  function removeFile(index: number) {
    uploadedFiles = uploadedFiles.filter((_, i) => i !== index);
  }

  function formatFileSize(bytes: number): string {
    if (bytes < 1024) return bytes + ' B';
    if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB';
    return (bytes / (1024 * 1024)).toFixed(1) + ' MB';
  }

  async function handleUploadSubmit() {
    if (uploadedFiles.length === 0) return;
    uploadLoading = true;
    await new Promise(r => setTimeout(r, 1500));
    uploadLoading = false;
    showUploadSheet = false;
    goto(`${base}/bank-check`);
  }

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

  function openOtpSheet() {
    if (!allValid) return;
    otp = ['', '', '', '', '', ''];
    otpError = '';
    showOtpSheet = true;
    startTimer();
    setTimeout(() => otpInputs[0]?.focus(), 300);
  }

  function handleOtpInput(i: number, e: Event) {
    const val = (e.target as HTMLInputElement).value.replace(/\D/g, '');
    if (!val) return;
    otp[i] = val.slice(-1);
    otpError = '';
    if (i < 5) otpInputs[i + 1]?.focus();
  }

  function handleOtpKeydown(i: number, e: KeyboardEvent) {
    if (e.key === 'Backspace') {
      if (otp[i]) { otp[i] = ''; }
      else if (i > 0) { otp[i - 1] = ''; otpInputs[i - 1]?.focus(); }
    }
  }

  function handleOtpPaste(e: ClipboardEvent) {
    e.preventDefault();
    const text = e.clipboardData?.getData('text').replace(/\D/g, '').slice(0, 6) ?? '';
    text.split('').forEach((ch, i) => { otp[i] = ch; });
    otpInputs[Math.min(text.length, 5)]?.focus();
  }

  let isOtpComplete = $derived(otp.every(d => d !== ''));

  // Bank fetch sheet
  let showBankSheet = $state(false);
  type BankFetchState = 'loading' | 'success' | 'accounts';
  let bankFetchState = $state<BankFetchState>('loading');

  const mockBankAccounts = [
    { id: 1, accountNumber: '************1100', branch: 'Bommanhalli branch', logo: 'bank-sbi.png' },
    { id: 2, accountNumber: '************2345', branch: 'Marathahalli branch', logo: 'bank-kotak.png' },
  ];

  let selectedBankId = $state<number | null>(null);

  async function handleOtpVerify() {
    if (!isOtpComplete) return;
    otpLoading = true;
    await new Promise(r => setTimeout(r, 600));
    otpLoading = false;
    showOtpSheet = false;
    clearInterval(timerInterval);

    // Open bank fetch sheet in loading state
    bankFetchState = 'loading';
    showBankSheet = true;

    // Simulate fetching (~3.5s so the rocket loader is clearly visible)
    await new Promise(r => setTimeout(r, 3500));
    bankFetchState = 'success';

    // Show success lottie long enough to complete (~2.5s)
    await new Promise(r => setTimeout(r, 2500));

    // Smart-animate effect: close the sheet, swap state, then reopen
    // so the BottomSheet's fly-up animation replays for the accounts view
    showBankSheet = false;
    await new Promise(r => setTimeout(r, 320)); // wait for out animation
    bankFetchState = 'accounts';
    await new Promise(r => setTimeout(r, 60));  // small breathing pause
    showBankSheet = true;
  }

  function handleResend() {
    if (!canResend) return;
    otp = ['', '', '', '', '', ''];
    otpError = '';
    startTimer();
    otpInputs[0]?.focus();
  }

  onDestroy(() => clearInterval(timerInterval));

  async function handleSubmit() {
    openMethodSheet();
  }
</script>

<svelte:head><title>Income details – IOB</title></svelte:head>

<div class="screen">

  <!-- ── HEADER ── -->
  <div class="header-area">
    <StatusBar />
    <div class="icon-row">
      <button class="icon-btn" aria-label="Back" onclick={() => goto(`${base}/pan`)}>
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
      <img src="{base}/income.svg" alt="Income" class="illustration-img" />
    </div>

    <div class="progress-row">
      <h1 class="screen-title">Enter income details</h1>
    </div>
  </div>

  <!-- ── FORM ── -->
  <div class="form-area">
    <ProgressiveInputStepper {steps} onchange={handleStepChange} mode="dependent" />
  </div>

  <!-- Bank Partner Logos -->
  <div class="partner-logos">
    <img src="{base}/logos-bank-partner.svg" alt="Bank Partners" class="partner-img" draggable="false" />
  </div>

  <div class="spacer"></div>

  <!-- ── FOOTER ── -->
  <div class="footer">
    <button
      class="btn-primary"
      disabled={!allValid || loading}
      onclick={handleSubmit}
    >
      {#if loading}
        <span class="spinner"></span>
      {:else}
        Continue
      {/if}
    </button>
  </div>

  <!-- Toast -->
  <div class="toast-position">
    <ToastMessage state="success" message="PAN successfully verified" visible={showToast} onclose={() => { showToast = false; }} />
  </div>

</div>

<!-- Bank Fetch Bottom Sheet -->
<BottomSheet bind:open={showBankSheet} title={bankFetchState === 'accounts' ? 'Continue to verify your income details' : ''}>

  {#if bankFetchState === 'loading' || bankFetchState === 'success'}
    <!-- Loading / Success card -->
    <div class="bank-status-card">
      {#if bankFetchState === 'loading'}
        <div class="bank-status-inner" in:fade={{ duration: 200 }}>
          <!-- Rocket loader (same as verification screen) -->
          <div class="bank-rocket-wrap">
            <div class="longfazers">
              <span></span><span></span><span></span><span></span>
            </div>
            <div class="loader-center">
              <div class="loader">
                <span>
                  <span></span><span></span><span></span><span></span>
                </span>
                <div class="base">
                  <span></span>
                  <div class="face"></div>
                </div>
              </div>
            </div>
          </div>
          <p class="bank-status-text">Fetching your bank details...</p>
        </div>
      {:else}
        <div class="bank-status-inner" in:fade={{ duration: 300 }}>
          {#if lottieReady}
            <dotlottie-wc
              src="{base}/success.lottie"
              autoplay
              speed="0.7"
              style="width: 80px; height: 80px;"
            ></dotlottie-wc>
          {/if}
          <p class="bank-status-text bank-status-success">Bank details fetched!</p>
        </div>
      {/if}
    </div>

  {:else if bankFetchState === 'accounts'}
    <!-- Account list -->
    <div class="bank-accounts-list" in:fade={{ duration: 200 }}>

      <!-- Section header -->
      <div class="bank-section-header">
        <div class="bank-section-line"></div>
        <span class="bank-section-label">Your bank accounts</span>
        <div class="bank-section-line"></div>
      </div>

      <!-- Accounts container + info bar wrapped together -->
      <div class="bank-accounts-wrapper">
        <div class="bank-accounts-container">
          {#each mockBankAccounts as account, i}
            <button
              class="bank-account-item"
              class:bank-account-selected={selectedBankId === account.id}
              onclick={() => selectedBankId = account.id}
            >
              <div class="bank-account-info">
                <p class="bank-account-number">{account.accountNumber}</p>
                <p class="bank-account-branch">{account.branch}</p>
              </div>
              <img src="{base}/{account.logo}" alt="Bank logo" class="bank-logo-img" />
            </button>
            {#if i < mockBankAccounts.length - 1}
              <div class="bank-divider"></div>
            {/if}
          {/each}
        </div>

        <!-- Info bar -->
        <div class="bank-info-bar">
          <p class="bank-info-text">We are verifying your income details to give you the best credit limit</p>
        </div>
      </div>

    </div>
  {/if}

  {#snippet footer()}
    {#if bankFetchState === 'accounts'}
      <button
        class="bank-confirm-btn"
        disabled={false}
        onclick={() => goto(`${base}/bank-check`)}
        in:fade={{ duration: 150 }}
      >
        Continue
      </button>
    {/if}
  {/snippet}
</BottomSheet>
<BottomSheet bind:open={showOtpSheet} title="Enter OTP">
  <div class="otp-sheet-content">
    <p class="otp-subtitle">Enter OTP sent to your mobile number</p>

    <div class="otp-row" onpaste={handleOtpPaste}>
      {#each otp as digit, i}
        <div class="otp-box" class:filled={!!digit} class:has-error={!!otpError}>
          <input
            bind:this={otpInputs[i]}
            type="tel"
            inputmode="numeric"
            maxlength="1"
            value={digit}
            oninput={(e) => handleOtpInput(i, e)}
            onkeydown={(e) => handleOtpKeydown(i, e)}
            class="otp-input"
            aria-label="OTP digit {i + 1}"
          />
        </div>
      {/each}
    </div>

    {#if otpError}
      <p class="otp-error" in:fade={{ duration: 150 }}>{otpError}</p>
    {/if}

    <div class="resend-row">
      {#if canResend}
        <button class="resend-btn" onclick={handleResend}>
          Not received? <span class="resend-link">Resend OTP</span>
        </button>
      {:else}
        <p class="resend-text">Resend OTP in {resendSeconds}s</p>
      {/if}
    </div>
  </div>

  {#snippet footer()}
    <button
      class="otp-verify-btn"
      disabled={!isOtpComplete || otpLoading}
      onclick={handleOtpVerify}
    >
      {#if otpLoading}
        <span class="spinner"></span>
      {:else}
        Continue
      {/if}
    </button>
  {/snippet}
</BottomSheet>

<!-- Verification Method Choice Sheet -->
<BottomSheet bind:open={showMethodSheet} title="Select a method to verify your income">
  <div class="method-sheet-content">
    <div class="method-options">
      <!-- Account Aggregator Option (Recommended) -->
      <button
        class="method-option"
        class:method-selected={selectedMethod === 'aggregator'}
        onclick={() => selectedMethod = 'aggregator'}
      >
        <div class="method-option-left">
          <div class="radio" class:checked={selectedMethod === 'aggregator'}>
            {#if selectedMethod === 'aggregator'}
              <div class="check-anim">
                <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                  <circle cx="10" cy="10" r="10" fill="#15803D"/>
                  <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </div>
            {/if}
          </div>
          <div class="method-info">
            <div class="method-title-row">
              <span class="method-title">Account Aggregator</span>
              <span class="method-tag">Recommended</span>
            </div>
            <p class="method-desc">Securely fetch your bank statements instantly via RBI-licensed aggregator</p>
          </div>
        </div>
      </button>

      <!-- Upload Statement Option -->
      <button
        class="method-option"
        class:method-selected={selectedMethod === 'upload'}
        onclick={() => selectedMethod = 'upload'}
      >
        <div class="method-option-left">
          <div class="radio" class:checked={selectedMethod === 'upload'}>
            {#if selectedMethod === 'upload'}
              <div class="check-anim">
                <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                  <circle cx="10" cy="10" r="10" fill="#15803D"/>
                  <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </div>
            {/if}
          </div>
          <div class="method-info">
            <span class="method-title">Upload bank statement</span>
            <p class="method-desc">Upload your last 3 months bank statement (PDF format)</p>
          </div>
        </div>
        <i class="ph ph-upload-simple" style="font-size:24px; color:#111827"></i>
      </button>
    </div>
  </div>

  {#snippet footer()}
    <!-- AA Consent (only shown when Account Aggregator is selected) -->
    {#if selectedMethod === 'aggregator'}
      <div class="method-consent-row">
        <button class="checkbox" class:checked={aaAgreed} onclick={() => aaAgreed = !aaAgreed} aria-label="Agree to AA terms">
          {#if aaAgreed}
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
        </button>
        <p class="method-consent-text">I authorize IOB Bank to verify my income details via a secure RBI-licensed Account Aggregator.</p>
      </div>
    {/if}
    <button
      class="method-continue-btn"
      disabled={!selectedMethod || (selectedMethod === 'aggregator' && !aaAgreed)}
      onclick={handleMethodContinue}
    >
      {selectedMethod === 'aggregator' ? 'Confirm with OTP' : 'Continue'}
    </button>
  {/snippet}
</BottomSheet>

<!-- Upload Statement Sheet -->
<BottomSheet bind:open={showUploadSheet} title="Upload bank statement">
  <div class="upload-sheet-content">
    <p class="upload-subtitle">Upload your last 3 months bank statement in PDF format</p>

    <!-- Upload Area -->
    <input
      type="file"
      accept=".pdf"
      multiple
      bind:this={fileInput}
      onchange={handleFileSelect}
      class="file-input-hidden"
    />
    
    <button class="upload-area" onclick={() => fileInput?.click()}>
      <div class="upload-icon-wrap">
        <i class="ph ph-cloud-arrow-up" style="font-size:32px; color:#184595"></i>
      </div>
      <p class="upload-area-text">Tap to upload PDF</p>
      <p class="upload-area-hint">Maximum 3 files, 10MB each</p>
    </button>

    <!-- Uploaded Files List -->
    {#if uploadedFiles.length > 0}
      <div class="uploaded-files">
        {#each uploadedFiles as file, i}
          <div class="uploaded-file-item">
            <div class="file-icon">
              <i class="ph ph-file-pdf" style="font-size:24px; color:#DC2626"></i>
            </div>
            <div class="file-info">
              <p class="file-name">{file.name}</p>
              <p class="file-size">{formatFileSize(file.size)}</p>
            </div>
            <button class="file-remove" onclick={() => removeFile(i)} aria-label="Remove file">
              <i class="ph ph-x" style="font-size:18px; color:#6B7280"></i>
            </button>
          </div>
        {/each}
      </div>
    {/if}
  </div>

  {#snippet footer()}
    <button
      class="upload-submit-btn"
      disabled={uploadedFiles.length === 0 || uploadLoading}
      onclick={handleUploadSubmit}
    >
      {#if uploadLoading}
        <span class="spinner"></span>
      {:else}
        Submit documents
      {/if}
    </button>
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

  .toast-position {
    position: fixed;
    bottom: 120px;
    left: 16px;
    right: 16px;
    z-index: 100;
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

  .progress-row {
    display: flex; align-items: flex-end; justify-content: space-between;
    padding: 16px 16px 0 16px; gap: 16px;
  }
  .screen-title {
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 24px;
    line-height: 1.2; color: #111827; letter-spacing: -0.25px; flex: 1;
  }

  /* ── Form ── */
  .form-area {
    display: flex;
    flex-direction: column;
    gap: 16px;
    padding: 20px 16px 0;
    flex-shrink: 0;
  }

  /* ── Dropdown field ── */
  .dropdown-field {
    display: flex;
    align-items: center;
    gap: 16px;
    background: #FFFCF4;
    border: 1px solid #D1D5DB;
    border-radius: 8px;
    padding: 10px 12px;
    min-height: 52px;
    cursor: pointer;
    transition: border-color 0.15s, background 0.15s, opacity 0.15s;
  }
  .dropdown-field.active {
    border-color: #242A80;
    background: #FFFCF4;
  }
  .dropdown-field.disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  .dropdown-content {
    display: flex;
    flex-direction: column;
    flex: 1;
    gap: 4px;
    min-width: 0;
    justify-content: center;
  }

  .dropdown-label {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 14px;
    line-height: 1.43;
    color: #6B7280;
    transition: font-size 0.15s ease;
  }
  .dropdown-label.floated {
    font-size: 12px;
    line-height: 1.5;
    color: #111827;
  }

  .dropdown-value {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    line-height: 1.43;
    color: #111827;
  }

  /* ── Option list in sheets ── */
  .option-list {
    display: flex;
    flex-direction: column;
    background: #FFFCF4;
    border-radius: 12px;
    overflow: hidden;
    border: 1px solid #F3F4F6;
  }

  .option-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 14px 12px;
    background: #FFFCF4;
    border: none;
    cursor: pointer;
    text-align: left;
    width: 100%;
    transition: background 0.12s;
  }
  .option-item:active { background: #FFFCF4; }

  .option-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 500;
    font-size: 14px;
    line-height: 1.43;
    color: #111827;
    flex: 1;
  }

  .option-radio {
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
  .option-radio.checked { border: none; background: none; }

  .option-divider {
    height: 1px;
    background: #F3F4F6;
    margin: 0 12px;
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

  /* ── Spacer ── */
  .spacer { flex: 1; }

  /* ── Partner Logos ── */
  .partner-logos {
    display: flex;
    justify-content: center;
    padding: 40px 16px 0;
  }
  .partner-img {
    height: 38px;
    filter: grayscale(100%);
    width: auto;
  }

  /* ── Footer ── */
  .footer { position: sticky; bottom: 0; z-index: 5; margin-top: auto;
    padding: 16px 16px calc(40px + env(safe-area-inset-bottom));
    background: #FFFCF4;
    flex-shrink: 0;
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  .btn-primary {
    display: flex; align-items: center; justify-content: center;
    width: 100%; height: 48px; background: #184595; color: #FFFFFF;
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 16px;
    border: none; border-radius: 8px; cursor: pointer;
    box-shadow: 0px 4px 0px #06142A;
    transition: opacity 0.15s, transform 0.1s;
  }
  .btn-primary:active:not(:disabled) { opacity: 0.88; transform: scale(0.99); }
  .btn-primary:disabled { background: #D1D5DB; box-shadow: none; cursor: not-allowed; }

  /* Consent */
  .consent-row {
    display: flex;
    align-items: flex-start;
    gap: 8px;
  }
  .checkbox {
    width: 24px; height: 24px; background: none; border: none;
    cursor: pointer; padding: 3px; flex-shrink: 0;
    display: flex; align-items: center; justify-content: center;
  }
  .check-anim {
    animation: checkPop 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
    display: flex;
  }
  @keyframes checkPop {
    0% { transform: scale(0); opacity: 0; }
    60% { transform: scale(1.15); opacity: 1; }
    100% { transform: scale(1); }
  }
  .consent-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 1.5;
    color: #6B7280;
  }

  .spinner {
    width: 20px; height: 20px;
    border: 2.5px solid rgba(255,255,255,0.4); border-top-color: #FFFFFF;
    border-radius: 50%; animation: spin 0.7s linear infinite;
  }
  @keyframes spin { to { transform: rotate(360deg); } }

  /* OTP Sheet */
  .otp-sheet-content {
    display: flex; flex-direction: column; gap: 0;
  }

  .otp-subtitle {
    font-family: 'Nunito Sans', sans-serif; font-weight: 400; font-size: 12px;
    color: #6B7280; line-height: 1.5; text-align: left;
    margin-top: -20px; margin-bottom: 24px;
  }

  .otp-row {
    display: flex; gap: 8px; width: 100%;
  }

  .otp-box {
    flex: 1; height: 52px; background: #FFFFFF;
    border: 1px solid #D1D5DB; border-radius: 8px;
    display: flex; align-items: center; justify-content: center;
  }
  .otp-box.filled { border-color: #D1D5DB; }
  .otp-box.has-error { border-color: #B91C1C; }

  .otp-input {
    width: 100%; height: 100%; background: none; border: none; outline: none;
    text-align: center; font-family: 'Nunito Sans', sans-serif;
    font-weight: 600; font-size: 16px; color: #111827; caret-color: #184595;
  }

  .resend-row {
    display: flex; justify-content: center; align-items: center;
    margin-top: 16px;
  }

  .resend-text {
    font-family: 'Nunito Sans', sans-serif; font-weight: 400;
    font-size: 13px; color: #6B7280;
  }

  .resend-btn {
    background: none; border: none; cursor: pointer;
    font-family: 'Nunito Sans', sans-serif; font-weight: 400;
    font-size: 13px; color: #6B7280;
  }

  .resend-link {
    color: #184595; font-weight: 600; text-decoration: underline;
  }

  .otp-error {
    font-family: 'Nunito Sans', sans-serif; font-size: 12px;
    color: #B91C1C; text-align: center;
  }

  .otp-verify-btn {
    display: flex; align-items: center; justify-content: center;
    width: 100%; height: 48px; background: #184595; color: #FFFFFF;
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 16px;
    border: none; border-radius: 8px; cursor: pointer;
    box-shadow: 0px 4px 0px #06142A; transition: opacity 0.15s, transform 0.1s;
  }
  .otp-verify-btn:active:not(:disabled) { opacity: 0.88; transform: scale(0.99); }
  .otp-verify-btn:disabled { background: #D1D5DB; box-shadow: none; cursor: not-allowed; }

  /* ── Bank Fetch Sheet ── */
  .bank-status-card {
    background: #FFFFFF;
    border-radius: 16px;
    padding: 32px 24px;
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 160px;
    margin-bottom: 8px;
    position: relative;
  }

  .bank-status-inner {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 20px;
    width: 100%;
  }

  :global(.bank-rocket-wrap) {
    position: relative;
    width: 100%;
    height: 60px;
  }

  :global(.bank-rocket-wrap .longfazers) {
    position: absolute;
    inset: 0;
    overflow: hidden;
  }
  :global(.bank-rocket-wrap .longfazers span) {
    position: absolute;
    height: 2px;
    width: 20%;
    background: rgba(24, 69, 149, 0.15);
  }
  :global(.bank-rocket-wrap .longfazers span:nth-child(1)) { top: 20%; animation: lf 0.6s linear infinite; animation-delay: -5s; }
  :global(.bank-rocket-wrap .longfazers span:nth-child(2)) { top: 40%; animation: lf2 0.8s linear infinite; animation-delay: -1s; }
  :global(.bank-rocket-wrap .longfazers span:nth-child(3)) { top: 60%; animation: lf3 0.6s linear infinite; }
  :global(.bank-rocket-wrap .longfazers span:nth-child(4)) { top: 80%; animation: lf4 0.5s linear infinite; animation-delay: -3s; }

  @keyframes lf  { 0% { left: 200%; } 100% { left: -200%; opacity: 0; } }
  @keyframes lf2 { 0% { left: 200%; } 100% { left: -200%; opacity: 0; } }
  @keyframes lf3 { 0% { left: 200%; } 100% { left: -100%; opacity: 0; } }
  @keyframes lf4 { 0% { left: 200%; } 100% { left: -100%; opacity: 0; } }

  :global(.bank-rocket-wrap .loader-center) {
    width: 100%;
    height: 60px;
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  :global(.bank-rocket-wrap .loader) {
    position: absolute;
    left: 50%;
    top: 50%;
    margin-left: -65px;
    margin-top: -10px;
    animation: speeder 0.4s linear infinite;
  }
  :global(.bank-rocket-wrap .loader > span) {
    height: 5px; width: 35px; background: #184595;
    position: absolute; top: -19px; left: 60px;
    border-radius: 2px 10px 1px 0;
  }
  :global(.bank-rocket-wrap .base span) {
    position: absolute; width: 0; height: 0;
    border-top: 6px solid transparent;
    border-right: 100px solid #184595;
    border-bottom: 6px solid transparent;
  }
  :global(.bank-rocket-wrap .base span::before) {
    content: ""; height: 22px; width: 22px;
    border-radius: 50%; background: #184595;
    position: absolute; right: -110px; top: -16px;
  }
  :global(.bank-rocket-wrap .base span::after) {
    content: ""; position: absolute; width: 0; height: 0;
    border-top: 0 solid transparent;
    border-right: 55px solid #184595;
    border-bottom: 16px solid transparent;
    top: -16px; right: -98px;
  }
  :global(.bank-rocket-wrap .face) {
    position: absolute; height: 12px; width: 20px;
    background: #184595; border-radius: 20px 20px 0 0;
    transform: rotate(-40deg); right: -125px; top: -15px;
  }
  :global(.bank-rocket-wrap .face::after) {
    content: ""; height: 12px; width: 12px; background: #184595;
    right: 4px; top: 7px; position: absolute;
    transform: rotate(40deg); transform-origin: 50% 50%;
    border-radius: 0 0 0 2px;
  }
  :global(.bank-rocket-wrap .loader > span > span) {
    width: 30px; height: 1px; background: #184595;
    position: absolute; animation: fazer1 0.2s linear infinite;
  }
  :global(.bank-rocket-wrap .loader > span > span:nth-child(2)) { top: 3px; animation: fazer2 0.4s linear infinite; }
  :global(.bank-rocket-wrap .loader > span > span:nth-child(3)) { top: 1px; animation: fazer3 0.4s linear infinite; animation-delay: -1s; }
  :global(.bank-rocket-wrap .loader > span > span:nth-child(4)) { top: 4px; animation: fazer4 1s linear infinite; animation-delay: -1s; }

  @keyframes fazer1 { 0% { left: 0; } 100% { left: -80px; opacity: 0; } }
  @keyframes fazer2 { 0% { left: 0; } 100% { left: -100px; opacity: 0; } }
  @keyframes fazer3 { 0% { left: 0; } 100% { left: -50px; opacity: 0; } }
  @keyframes fazer4 { 0% { left: 0; } 100% { left: -150px; opacity: 0; } }
  @keyframes speeder {
    0%   { transform: translate(2px, 1px) rotate(0deg); }
    10%  { transform: translate(-1px, -3px) rotate(-1deg); }
    20%  { transform: translate(-2px, 0px) rotate(1deg); }
    30%  { transform: translate(1px, 2px) rotate(0deg); }
    40%  { transform: translate(1px, -1px) rotate(1deg); }
    50%  { transform: translate(-1px, 3px) rotate(-1deg); }
    60%  { transform: translate(-1px, 1px) rotate(0deg); }
    70%  { transform: translate(3px, 1px) rotate(-1deg); }
    80%  { transform: translate(-2px, -1px) rotate(1deg); }
    90%  { transform: translate(2px, 1px) rotate(0deg); }
    100% { transform: translate(1px, -2px) rotate(-1deg); }
  }

  .bank-status-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 500;
    font-size: 14px;
    color: #111827;
    text-align: center;
  }

  .bank-status-success {
    color: #111827;
    font-weight: 600;
  }

  /* ── Bank Account List ── */
  .bank-accounts-list {
    display: flex;
    flex-direction: column;
    gap: 16px;
    margin-bottom: 4px;
  }

  /* Section header with gradient lines */
  .bank-section-header {
    display: flex;
    align-items: center;
    gap: 10px;
  }
  .bank-section-line {
    flex: 1;
    height: 1px;
    background: linear-gradient(90deg, #A3ABBB 0%, rgba(22, 23, 26, 0) 100%);
    transform: scaleX(-1);
  }
  .bank-section-line:last-child {
    background: linear-gradient(270deg, #A3ABBB 0%, rgba(22, 23, 26, 0) 100%);
    transform: scaleX(-1);
  }
  .bank-section-label {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 12px;
    line-height: 18px;
    color: #6B7280;
    white-space: nowrap;
  }

  /* Accounts + info bar wrapper */
  .bank-accounts-wrapper {
    display: flex;
    flex-direction: column;
    gap: 0;
    border: 0.5px solid #D1D5DB;
    border-radius: 12px;
    overflow: hidden;
  }

  /* Accounts container with border */
  .bank-accounts-container {
    background: #FFFFFF;
  }

  .bank-account-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 8px;
    padding: 10px 12px;
    background: #FFFFFF;
    border: none;
    cursor: pointer;
    text-align: left;
    width: 100%;
    transition: background 0.12s;
  }
  .bank-account-item:active { background: #F9FAFB; }
  .bank-account-selected { background: #F0F4FF; }

  .bank-account-info {
    display: flex;
    flex-direction: column;
    gap: 4px;
    flex: 1;
    min-width: 0;
  }

  .bank-account-number {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    line-height: 20px;
    color: #111827;
  }

  .bank-account-branch {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 18px;
    color: #6B7280;
  }

  .bank-logo-img {
    height: 24px;
    width: auto;
    object-fit: contain;
    flex-shrink: 0;
  }

  .bank-divider {
    height: 0.5px;
    background: #D1D5DB;
    margin: 0 12px;
  }

  /* Info bar */
  .bank-info-bar {
    background: #F0FDF4;
    padding: 10px;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .bank-info-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 12px;
    line-height: 18px;
    color: #15803D;
    text-align: center;
  }

  .bank-confirm-btn {
    display: flex; align-items: center; justify-content: center;
    width: 100%; height: 48px; background: #184595; color: #FFFFFF;
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 16px;
    border: none; border-radius: 8px; cursor: pointer;
    box-shadow: 0px 4px 0px #06142A; transition: opacity 0.15s, transform 0.1s;
  }
  .bank-confirm-btn:active:not(:disabled) { opacity: 0.88; transform: scale(0.99); }
  .bank-confirm-btn:disabled { background: #D1D5DB; box-shadow: none; cursor: not-allowed; }

  /* ── Method Choice Sheet ── */
  .method-sheet-content {
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  .method-subtitle {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 14px;
    color: #6B7280;
    line-height: 1.5;
    margin-top: -20px;
    margin-bottom: 8px;
  }

  .method-options {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  .method-option {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
    padding: 16px;
    background: #FFFFFF;
    border: 1px solid #D1D5DB;
    border-radius: 12px;
    cursor: pointer;
    text-align: left;
    width: 100%;
    transition: border-color 0.15s, background 0.15s;
  }
  .method-option:active { background: #F9FAFB; }
  .method-selected {
    border-color: #184595;
    background: #F0F4FF;
  }

  .method-option-left {
    display: flex;
    align-items: flex-start;
    gap: 12px;
    flex: 1;
    min-width: 0;
  }

  .method-option .radio {
    margin-top: 2px;
  }

  .method-info {
    display: flex;
    flex-direction: column;
    gap: 4px;
    flex: 1;
    min-width: 0;
  }

  .method-title-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 8px;
  }

  .method-title {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    line-height: 1.4;
    color: #111827;
  }

  .method-tag {
    display: inline-flex;
    align-items: center;
    padding: 4px 8px;
    background: #F0FDF4;
    border-radius: 4px;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 18px;
    color: #15803D;
  }

  .method-desc {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 1.5;
    color: #6B7280;
  }

  .method-continue-btn {
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
  .method-continue-btn:active:not(:disabled) { opacity: 0.88; transform: scale(0.99); }
  .method-continue-btn:disabled { background: #D1D5DB; box-shadow: none; cursor: not-allowed; }

  /* AA Consent in Method Sheet */
  .method-consent-row {
    display: flex;
    align-items: flex-start;
    gap: 8px;
    margin-bottom: 16px;
  }
  .method-consent-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 1.5;
    color: #6B7280;
    flex: 1;
  }

  /* ── Upload Sheet ── */
  .upload-sheet-content {
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  .upload-subtitle {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 14px;
    color: #6B7280;
    line-height: 1.5;
    margin-top: -20px;
    margin-bottom: 8px;
  }

  .file-input-hidden {
    position: absolute;
    opacity: 0;
    width: 0;
    height: 0;
    pointer-events: none;
  }

  .upload-area {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 8px;
    padding: 32px 24px;
    background: #FFFFFF;
    border: 2px dashed #D1D5DB;
    border-radius: 12px;
    cursor: pointer;
    transition: border-color 0.15s, background 0.15s;
  }
  .upload-area:active {
    background: #F9FAFB;
    border-color: #184595;
  }

  .upload-icon-wrap {
    width: 56px;
    height: 56px;
    border-radius: 50%;
    background: #F0F4FF;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .upload-area-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    color: #111827;
  }

  .upload-area-hint {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    color: #6B7280;
  }

  .uploaded-files {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .uploaded-file-item {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 12px;
    background: #FFFFFF;
    border: 1px solid #D1D5DB;
    border-radius: 8px;
  }

  .file-icon {
    width: 40px;
    height: 40px;
    border-radius: 8px;
    background: #FEF2F2;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }

  .file-info {
    display: flex;
    flex-direction: column;
    gap: 2px;
    flex: 1;
    min-width: 0;
  }

  .file-name {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 13px;
    color: #111827;
    line-height: 1.4;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .file-size {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 11px;
    color: #6B7280;
  }

  .file-remove {
    width: 32px;
    height: 32px;
    border-radius: 50%;
    background: none;
    border: none;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    transition: background 0.15s;
  }
  .file-remove:active { background: #F3F4F6; }

  .upload-submit-btn {
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
  .upload-submit-btn:active:not(:disabled) { opacity: 0.88; transform: scale(0.99); }
  .upload-submit-btn:disabled { background: #D1D5DB; box-shadow: none; cursor: not-allowed; }
</style>
