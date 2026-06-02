<script lang="ts">
  import { goto } from '$app/navigation';
  import { tick } from 'svelte';
  import StatusBar from '$lib/components/StatusBar.svelte';
  import ProgressComponent from '$lib/components/ProgressComponent.svelte';
  import BottomSheet from '$lib/components/BottomSheet.svelte';

  // Field values
  let occupation = $state('');
  let grossIncome = $state('');
  let sourceOfIncome = $state('');

  // Sheet visibility
  let showOccupationSheet = $state(false);
  let showIncomeSheet = $state(false);
  let showSourceSheet = $state(false);

  let loading = $state(false);

  // Options
  const occupationOptions = [
    'Salaried',
    'Self-employed',
    'Business owner',
    'Freelancer / Consultant',
    'Retired',
    'Student',
    'Homemaker',
    'Other',
  ];

  const incomeOptions = [
    'Below ₹2.5 lakhs',
    '₹2.5 to 5 lakhs',
    '₹5 to 10 lakhs',
    '₹10 to 25 lakhs',
    '₹25 to 50 lakhs',
    'Above ₹50 lakhs',
  ];

  const sourceOptions = [
    'Salary',
    'Business income',
    'Rental income',
    'Investment returns',
    'Pension',
    'Agriculture',
    'Other',
  ];

  // Cascade: select occupation → auto-open income sheet
  async function selectOccupation(val: string) {
    occupation = val;
    showOccupationSheet = false;
    await tick();
    setTimeout(() => { showIncomeSheet = true; }, 320);
  }

  // Cascade: select income → auto-open source sheet
  async function selectIncome(val: string) {
    grossIncome = val;
    showIncomeSheet = false;
    await tick();
    setTimeout(() => { showSourceSheet = true; }, 320);
  }

  function selectSource(val: string) {
    sourceOfIncome = val;
    showSourceSheet = false;
  }

  let allValid = $derived(
    occupation.length > 0 &&
    grossIncome.length > 0 &&
    sourceOfIncome.length > 0
  );

  async function handleSubmit() {
    if (!allValid) return;
    loading = true;
    await new Promise(r => setTimeout(r, 600));
    loading = false;
    goto('/address');
  }
</script>

<svelte:head><title>Income details – DCB</title></svelte:head>

<div class="screen">

  <!-- ── HEADER ── -->
  <div class="header-area">
    <StatusBar />
    <div class="icon-row">
      <button class="icon-btn" aria-label="Back" onclick={() => goto('/personal-details')}>
        <i class="ph ph-caret-left"></i>
      </button>
      <div class="icon-group">
        <button class="icon-btn" aria-label="Help">
          <i class="ph-duotone ph-headset" style="font-size:24px"></i>
        </button>
      </div>
    </div>
    <div class="progress-row">
      <h1 class="screen-title">Enter income details</h1>
      <ProgressComponent step={7} totalSteps={9} />
    </div>
  </div>

  <!-- ── FORM ── -->
  <div class="form-area">

    <!-- Occupation -->
    <!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
    <div
      class="dropdown-field"
      class:active={occupation.length > 0}
      onclick={() => showOccupationSheet = true}
      role="button"
      tabindex="0"
    >
      <div class="dropdown-content">
        <span class="dropdown-label" class:floated={occupation.length > 0}>Your occupation</span>
        {#if occupation.length > 0}
          <span class="dropdown-value">{occupation}</span>
        {/if}
      </div>
      <i class="ph ph-caret-down" style="font-size:20px; color:#242A80; flex-shrink:0"></i>
    </div>

    <!-- Gross annual income -->
    <!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
    <div
      class="dropdown-field"
      class:active={grossIncome.length > 0}
      class:disabled={occupation.length === 0}
      onclick={() => { if (occupation.length > 0) showIncomeSheet = true; }}
      role="button"
      tabindex="0"
    >
      <div class="dropdown-content">
        <span class="dropdown-label" class:floated={grossIncome.length > 0}>Gross annual income</span>
        {#if grossIncome.length > 0}
          <span class="dropdown-value">{grossIncome}</span>
        {/if}
      </div>
      <i class="ph ph-caret-down" style="font-size:20px; color:{occupation.length > 0 ? '#242A80' : '#D1D5DB'}; flex-shrink:0"></i>
    </div>

    <!-- Primary source of income -->
    <!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
    <div
      class="dropdown-field"
      class:active={sourceOfIncome.length > 0}
      class:disabled={grossIncome.length === 0}
      onclick={() => { if (grossIncome.length > 0) showSourceSheet = true; }}
      role="button"
      tabindex="0"
    >
      <div class="dropdown-content">
        <span class="dropdown-label" class:floated={sourceOfIncome.length > 0}>Primary source of income</span>
        {#if sourceOfIncome.length > 0}
          <span class="dropdown-value">{sourceOfIncome}</span>
        {/if}
      </div>
      <i class="ph ph-caret-down" style="font-size:20px; color:{grossIncome.length > 0 ? '#242A80' : '#D1D5DB'}; flex-shrink:0"></i>
    </div>

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

</div>

<!-- ── OCCUPATION SHEET ── -->
<BottomSheet bind:open={showOccupationSheet} title="Select your occupation">
  <div class="option-list">
    {#each occupationOptions as opt, i}
      <button class="option-item" onclick={() => selectOccupation(opt)}>
        <span class="option-text">{opt}</span>
        <div class="option-radio" class:checked={occupation === opt}>
          {#if occupation === opt}
            <div class="check-anim">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <circle cx="10" cy="10" r="10" fill="#15803D"/>
                <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </div>
          {/if}
        </div>
      </button>
      {#if i < occupationOptions.length - 1}
        <div class="option-divider"></div>
      {/if}
    {/each}
  </div>
</BottomSheet>

<!-- ── GROSS INCOME SHEET ── -->
<BottomSheet bind:open={showIncomeSheet} title="Select gross annual income">
  <div class="option-list">
    {#each incomeOptions as opt, i}
      <button class="option-item" onclick={() => selectIncome(opt)}>
        <span class="option-text">{opt}</span>
        <div class="option-radio" class:checked={grossIncome === opt}>
          {#if grossIncome === opt}
            <div class="check-anim">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <circle cx="10" cy="10" r="10" fill="#15803D"/>
                <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </div>
          {/if}
        </div>
      </button>
      {#if i < incomeOptions.length - 1}
        <div class="option-divider"></div>
      {/if}
    {/each}
  </div>
</BottomSheet>

<!-- ── SOURCE OF INCOME SHEET ── -->
<BottomSheet bind:open={showSourceSheet} title="Select source of income">
  <div class="option-list">
    {#each sourceOptions as opt, i}
      <button class="option-item" onclick={() => selectSource(opt)}>
        <span class="option-text">{opt}</span>
        <div class="option-radio" class:checked={sourceOfIncome === opt}>
          {#if sourceOfIncome === opt}
            <div class="check-anim">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <circle cx="10" cy="10" r="10" fill="#15803D"/>
                <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </div>
          {/if}
        </div>
      </button>
      {#if i < sourceOptions.length - 1}
        <div class="option-divider"></div>
      {/if}
    {/each}
  </div>
</BottomSheet>

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

  .icon-row { display: flex; align-items: center; justify-content: space-between; width: 100%; }
  .icon-group { display: flex; align-items: center; }
  .icon-btn {
    display: flex; align-items: center; justify-content: center;
    width: 48px; height: 48px; background: none; border: none;
    cursor: pointer; border-radius: 50%; transition: background 0.15s;
  }
  .icon-btn:active { background: rgba(0,0,0,0.06); }

  .progress-row {
    display: flex; align-items: flex-end; justify-content: space-between;
    padding: 0 16px; gap: 16px;
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
    background: #FFFDF6;
    border: 1px solid #D1D5DB;
    border-radius: 8px;
    padding: 10px 12px;
    min-height: 52px;
    cursor: pointer;
    transition: border-color 0.15s, background 0.15s, opacity 0.15s;
  }
  .dropdown-field.active {
    border-color: #242A80;
    background: #FFFDF6;
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
    background: #FFFDF6;
    border-radius: 12px;
    overflow: hidden;
    border: 1px solid #F3F4F6;
  }

  .option-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 14px 12px;
    background: #FFFDF6;
    border: none;
    cursor: pointer;
    text-align: left;
    width: 100%;
    transition: background 0.12s;
  }
  .option-item:active { background: #FFFDF6; }

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
    background: #FFFDF6;
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
    background: #FFFDF6;
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
