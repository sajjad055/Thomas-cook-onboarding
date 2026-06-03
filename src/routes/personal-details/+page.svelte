<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import StatusBar from '$lib/components/StatusBar.svelte';
  import TextField from '$lib/components/TextField.svelte';
  import BottomSheet from '$lib/components/BottomSheet.svelte';

  // Marital status
  let maritalStatus = $state<'married' | 'single' | null>('single');

  // Father / Spouse toggle
  let nameOf = $state<'father' | 'spouse'>('father');

  // Field values
  let parentName = $state('');
  let motherName = $state('');
  let netWorth = $state('');

  let loading = $state(false);
  let showNetWorthSheet = $state(false);

  const netWorthOptions = [
    '₹2 to 5 lakhs',
    '₹5 to 10 lakhs',
    '₹10 to 25 lakhs',
    '₹25 to 50 lakhs',
    '₹50 lakhs to 1 crore',
    'Above ₹1 crore',
  ];

  function selectNetWorth(val: string) {
    netWorth = val;
    showNetWorthSheet = false;
  }

  let allValid = $derived(
    maritalStatus !== null &&
    parentName.trim().length > 0 &&
    motherName.trim().length > 0
  );

  async function handleSubmit() {
    if (!allValid) return;
    loading = true;
    await new Promise(r => setTimeout(r, 600));
    loading = false;
    goto(`${base}/income`);
  }
</script>

<svelte:head><title>Personal details – IOB</title></svelte:head>

<div class="screen">

  <!-- ── HEADER ── -->
  <div class="header-area">
    <StatusBar />
    <div class="icon-row">
      <button class="icon-btn" aria-label="Back" onclick={() => goto(`${base}/card-name`)}>
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
      <img src="{base}/security.svg" alt="" class="illustration-img" />
    </div>

    <div class="progress-row">
      <h1 class="screen-title">Enter personal details</h1>
    </div>
  </div>

  <!-- ── FORM ── -->
  <div class="form-area">

    <!-- Marital status -->
    <div class="section marital-section">
      <div class="marital-row">

        <!-- Married -->
        <button
          class="marital-option"
          class:selected={maritalStatus === 'married'}
          onclick={() => maritalStatus = 'married'}
        >
          <div class="radio-check" class:checked={maritalStatus === 'married'}>
            {#if maritalStatus === 'married'}
              <div class="check-anim">
                <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                  <circle cx="10" cy="10" r="10" fill="#15803D"/>
                  <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </div>
            {/if}
          </div>
          <span class="marital-label">I am Married</span>
        </button>

        <!-- Single -->
        <button
          class="marital-option"
          class:selected={maritalStatus === 'single'}
          onclick={() => maritalStatus = 'single'}
        >
          <div class="radio-check" class:checked={maritalStatus === 'single'}>
            {#if maritalStatus === 'single'}
              <div class="check-anim">
                <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                  <circle cx="10" cy="10" r="10" fill="#15803D"/>
                  <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </div>
            {/if}
          </div>
          <span class="marital-label">I am Single</span>
        </button>

      </div>
    </div>

    <!-- Father's name -->
    <div class="section">
      <TextField
        label="Father's name"
        bind:value={parentName}
        placeholder="Enter name"
      />
    </div>

    <!-- Mother's maiden name -->
    <div class="section">
      <TextField
        label="Mother's maiden name"
        bind:value={motherName}
        placeholder="Enter name"
        supportText="What is mother's maiden name? <a href='#'>Know more</a>"
      />
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

<!-- ── NET WORTH BOTTOM SHEET ── -->
<BottomSheet bind:open={showNetWorthSheet} title="Select your net worth">
  <div class="nw-list">
    {#each netWorthOptions as opt, i}
      <button
        class="nw-item"
        onclick={() => selectNetWorth(opt)}
      >
        <span class="nw-text">{opt}</span>
        <div class="nw-radio" class:checked={netWorth === opt}>
          {#if netWorth === opt}
            <div class="check-anim">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <circle cx="10" cy="10" r="10" fill="#15803D"/>
                <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </div>
          {/if}
        </div>
      </button>
      {#if i < netWorthOptions.length - 1}
        <div class="nw-divider"></div>
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
    background: #FFFCF4;
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
    display: flex; align-items: center; justify-content: space-between; width: 100%;
  }
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
    padding: 24px 0 0;
    flex-shrink: 0;
  }

  .section {
    padding: 0 16px;
  }

  .section.marital-section {
    margin-bottom: 8px;
  }

  /* ── Marital status ── */
  .marital-row {
    display: flex;
    align-items: center;
    gap: 24px;
  }

  .marital-option {
    display: flex;
    align-items: center;
    gap: 4px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;
  }

  .radio-check {
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
  .radio-check.checked { border: none; background: none; }

  .check-anim {
    animation: checkPop 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
    display: flex;
  }
  @keyframes checkPop {
    0%   { transform: scale(0); opacity: 0; }
    60%  { transform: scale(1.15); opacity: 1; }
    100% { transform: scale(1); }
  }

  .marital-label {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    line-height: 1.43;
    color: #111827;
  }

  /* ── Father/Spouse field group ── */
  .field-group {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .toggle-label-row {
    display: flex;
    align-items: center;
    gap: 8px;
    flex-wrap: wrap;
  }

  .toggle-prefix {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 1.5;
    color: #6B7280;
    white-space: nowrap;
  }

  .toggle-pills {
    display: flex;
    align-items: center;
    gap: 4px;
  }

  .toggle-pill {
    padding: 4px 8px;
    border-radius: 500px;
    border: 1px solid #D1D5DB;
    background: #FFFCF4;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 18px;
    color: #111827;
    cursor: pointer;
    transition: all 0.15s;
  }
  .toggle-pill.active {
    background: #ECF2FF;
    border-color: #242A80;
    color: #242A80;
  }

  /* ── Net worth field trigger ── */
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
    transition: border-color 0.15s, background 0.15s;
  }
  .dropdown-field.active {
    border-color: #242A80;
    background: #FFFCF4;
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
    transition: font-size 0.18s ease;
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

  /* ── Net worth bottom sheet list ── */
  .nw-list {
    display: flex;
    flex-direction: column;
    background: #FFFCF4;
    border-radius: 12px;
    overflow: hidden;
    border: 1px solid #F3F4F6;
  }

  .nw-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 14px 12px;
    background: #FFFCF4;
    border: none;
    cursor: pointer;
    text-align: left;
    transition: background 0.12s;
    width: 100%;
  }
  .nw-item:active { background: #FFFCF4; }

  .nw-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 500;
    font-size: 14px;
    line-height: 1.43;
    color: #111827;
    flex: 1;
  }

  .nw-radio {
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
  .nw-radio.checked { border: none; background: none; }

  .nw-divider {
    height: 1px;
    background: #F3F4F6;
    margin: 0 12px;
  }

  /* check-anim used in net worth sheet radio */
  .check-anim {
    animation: checkPop 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
    display: flex;
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
