<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import StatusBar from '$lib/components/StatusBar.svelte';
  import ProgressiveInputStepper from '$lib/components/ProgressiveInputStepper.svelte';
  import ToastMessage from '$lib/components/ToastMessage.svelte';

  let showToast = $state(false);

  onMount(() => {
    showToast = true;
    setTimeout(() => { showToast = false; }, 2000);
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

  function handleStepChange(stepIndex: number, value: string) {
    steps[stepIndex].value = value;
    // Clear subsequent steps when a previous one changes
    for (let i = stepIndex + 1; i < steps.length; i++) {
      steps[i].value = '';
    }
  }

  let allValid = $derived(steps.every(s => s.value !== ''));

  async function handleSubmit() {
    if (!allValid) return;
    loading = true;
    await new Promise(r => setTimeout(r, 600));
    loading = false;
    goto(`${base}/verification`);
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
      <img src="{base}/security.svg" alt="Income" class="illustration-img" />
    </div>

    <div class="progress-row">
      <h1 class="screen-title">Enter income details</h1>
    </div>
  </div>

  <!-- ── FORM ── -->
  <div class="form-area">
    <ProgressiveInputStepper {steps} onchange={handleStepChange} />
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
        Submit & continue
      {/if}
    </button>
  </div>

  <!-- Toast -->
  <div class="toast-position">
    <ToastMessage state="success" message="PAN successfully verified" visible={showToast} onclose={() => { showToast = false; }} />
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

  /* ── Footer ── */
  .footer { position: sticky; bottom: 0; z-index: 5; margin-top: auto;
    padding: 16px;
    background: #FFFCF4;
    flex-shrink: 0;
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

  .spinner {
    width: 20px; height: 20px;
    border: 2.5px solid rgba(255,255,255,0.4); border-top-color: #FFFFFF;
    border-radius: 50%; animation: spin 0.7s linear infinite;
  }
  @keyframes spin { to { transform: rotate(360deg); } }
</style>
