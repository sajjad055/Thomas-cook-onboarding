<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import StatusBar from '$lib/components/StatusBar.svelte';
  import TextField from '$lib/components/TextField.svelte';
  import BottomSheet from '$lib/components/BottomSheet.svelte';

  let nomineeName = $state('');
  let relationship = $state('');
  let dob = $state('');
  let nomineeAddress = $state('Tanvi residency, Anugraha layout, Bilekahalli, Bangalore, Karnataka – 560076');
  let loading = $state(false);
  let showAddressSheet = $state(false);
  let showRelationshipSheet = $state(false);

  const relationshipOptions = [
    'Father',
    'Mother',
    'Spouse',
    'Brother',
    'Sister',
    'Son',
    'Daughter',
    'Other',
  ];

  const addressOptions = [
    'Tanvi residency, Anugraha layout, Bilekahalli, Bangalore, Karnataka – 560076',
    'G6, Perody Palace, Kodichiknahalli, Bommanahalli, Bangalore – 560076',
  ];

  function selectAddress(addr: string) {
    nomineeAddress = addr;
    showAddressSheet = false;
  }

  function selectRelationship(rel: string) {
    relationship = rel;
    showRelationshipSheet = false;
  }

  function onDobInput(e: Event) {
    let raw = (e.target as HTMLInputElement).value.replace(/\D/g, '').slice(0, 8);
    if (raw.length > 4) raw = raw.slice(0, 2) + '/' + raw.slice(2, 4) + '/' + raw.slice(4);
    else if (raw.length > 2) raw = raw.slice(0, 2) + '/' + raw.slice(2);
    dob = raw;
  }

  let allValid = $derived(
    nomineeName.trim().length > 0 &&
    relationship.trim().length > 0 &&
    dob.length >= 10 &&
    nomineeAddress.trim().length > 0
  );

  async function handleSubmit() {
    if (!allValid) return;
    loading = true;
    await new Promise(r => setTimeout(r, 600));
    loading = false;
    goto(`${base}/vkyc`);
  }
</script>

<svelte:head><title>Nominee details – IOB</title></svelte:head>

<div class="screen">

  <!-- ── HEADER ── -->
  <div class="header-area">
    <StatusBar />
    <div class="icon-row">
      <button class="icon-btn" aria-label="Back" onclick={() => goto(`${base}/address`)}>
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
      <img src="{base}/nominee.svg" alt="" class="illustration-img" />
    </div>

    <div class="progress-row">
      <h1 class="screen-title">Enter Nominee Details</h1>
    </div>
  </div>

  <!-- ── FORM ── -->
  <div class="form-area">

    <TextField
      label="Nominee's name"
      bind:value={nomineeName}
      placeholder="Enter full name"
    />

    <!-- Relationship dropdown -->
    <!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
    <div
      class="dropdown-field"
      class:active={relationship.length > 0}
      onclick={() => showRelationshipSheet = true}
      role="button"
      tabindex="0"
    >
      <div class="dropdown-content">
        <span class="dropdown-label" class:floated={relationship.length > 0}>Relationship with nominee</span>
        {#if relationship.length > 0}
          <span class="dropdown-value">{relationship}</span>
        {/if}
      </div>
      <i class="ph ph-caret-down" style="font-size:20px; color:#111827; flex-shrink:0"></i>
    </div>

    <TextField
      label="Nominee's date of birth"
      bind:value={dob}
      placeholder="DD/MM/YYYY"
      inputmode="numeric"
      maxlength={10}
      oninput={onDobInput}
    />

    <!-- Address dropdown -->
    <!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
    <div
      class="dropdown-field"
      class:active={nomineeAddress.length > 0}
      onclick={() => showAddressSheet = true}
      role="button"
      tabindex="0"
    >
      <div class="dropdown-content">
        <span class="dropdown-label" class:floated={nomineeAddress.length > 0}>Nominee's address</span>
        {#if nomineeAddress.length > 0}
          <span class="dropdown-value">{nomineeAddress}</span>
        {/if}
      </div>
      <i class="ph ph-caret-down" style="font-size:20px; color:#111827; flex-shrink:0"></i>
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
        Continue
      {/if}
    </button>
    <button class="btn-secondary" onclick={() => goto(`${base}/vkyc`)}>
      Skip adding nominee
    </button>
  </div>

</div>

<!-- Address Bottom Sheet -->
<BottomSheet bind:open={showAddressSheet} title="Select nominee's address">
  <div class="address-list">
    {#each addressOptions as addr, i}
      <button class="address-option" onclick={() => selectAddress(addr)}>
        <div class="radio" class:checked={nomineeAddress === addr}>
          {#if nomineeAddress === addr}
            <div class="check-anim">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <circle cx="10" cy="10" r="10" fill="#15803D"/>
                <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </div>
          {/if}
        </div>
        <p class="address-text">{addr}</p>
      </button>
      {#if i < addressOptions.length - 1}
        <div class="address-divider"></div>
      {/if}
    {/each}

    <!-- Dashed separator + Add address button -->
    <div class="dashed-divider"></div>
    <button class="add-address-btn">
      <i class="ph ph-plus" style="font-size:20px; color:#111827"></i>
      <span class="add-address-text">Add new address</span>
    </button>
  </div>
</BottomSheet>

<!-- Relationship Bottom Sheet -->
<BottomSheet bind:open={showRelationshipSheet} title="Select relationship">
  <div class="address-list">
    {#each relationshipOptions as rel, i}
      <button class="address-option" onclick={() => selectRelationship(rel)}>
        <div class="radio" class:checked={relationship === rel}>
          {#if relationship === rel}
            <div class="check-anim">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <circle cx="10" cy="10" r="10" fill="#15803D"/>
                <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </div>
          {/if}
        </div>
        <p class="address-text">{rel}</p>
      </button>
      {#if i < relationshipOptions.length - 1}
        <div class="address-divider"></div>
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
    width: 60px; height: 60px; border-radius: 50%;
    background: #F3F4F6; border: 0.5px solid #F5F5F5;
    margin-left: 16px; flex-shrink: 0;
    display: flex; align-items: center; justify-content: center; overflow: hidden;
  }
  .illustration-img { width: 100%; height: 100%; object-fit: cover; }

  .progress-row {
    display: flex; align-items: flex-end; justify-content: space-between;
    padding: 16px 16px 0 16px; gap: 16px;
  }
  .screen-title {
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 24px;
    line-height: 1.2; color: #111827; letter-spacing: -0.25px; flex: 1;
  }

  .form-area {
    display: flex; flex-direction: column; gap: 16px;
    padding: 24px 16px 0; flex-shrink: 0;
  }

  /* Dropdown field */
  .dropdown-field {
    display: flex; align-items: center; gap: 8px;
    background: #FFFFFF; border: 1px solid #D1D5DB; border-radius: 8px;
    padding: 12px; min-height: 52px; cursor: pointer;
  }
  .dropdown-content { flex: 1; display: flex; flex-direction: column; gap: 4px; }
  .dropdown-label {
    font-family: 'Nunito Sans', sans-serif; font-weight: 400; font-size: 12px;
    color: #6B7280; transition: color 0.15s;
  }
  .dropdown-label.floated { color: #111827; }
  .dropdown-value {
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 13px;
    color: #111827; line-height: 1.4;
    display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden;
  }

  .spacer { flex: 1; }

  .footer {
    position: sticky; bottom: 0; z-index: 5; margin-top: auto;
    padding: 20px 16px calc(40px + env(safe-area-inset-bottom));
    background: #FFFCF4; flex-shrink: 0;
    display: flex; flex-direction: column; gap: 12px;
  }

  .btn-primary {
    display: flex; align-items: center; justify-content: center;
    width: 100%; height: 48px; background: #184595; color: #FFFFFF;
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 16px;
    border: none; border-radius: 8px; cursor: pointer;
    box-shadow: 0px 4px 0px #06142A; transition: opacity 0.15s, transform 0.1s;
  }
  .btn-primary:active:not(:disabled) { opacity: 0.88; transform: scale(0.99); }
  .btn-primary:disabled { background: #D1D5DB; box-shadow: none; cursor: not-allowed; }

  .btn-secondary {
    display: flex; align-items: center; justify-content: center;
    width: 100%; height: 48px; background: transparent; color: #184595;
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 14px;
    border: none; border-radius: 8px; cursor: pointer;
    transition: opacity 0.15s;
  }
  .btn-secondary:active { opacity: 0.7; }

  .spinner {
    width: 20px; height: 20px;
    border: 2.5px solid rgba(255,255,255,0.4); border-top-color: #FFFFFF;
    border-radius: 50%; animation: spin 0.7s linear infinite;
  }
  @keyframes spin { to { transform: rotate(360deg); } }

  /* Address sheet */
  .address-list {
    display: flex; flex-direction: column;
    background: #FFFFFF;
    border-radius: 12px;
    overflow: hidden;
    border: 1px solid #F3F4F6;
  }
  .address-option {
    display: flex; align-items: flex-start; gap: 10px;
    padding: 12px 16px; background: none; border: none; cursor: pointer; text-align: left;
  }
  .radio {
    width: 20px; height: 20px; border-radius: 50%;
    border: 2px solid #D1D5DB; display: flex; align-items: center;
    justify-content: center; flex-shrink: 0; background: transparent;
  }
  .radio.checked { border: none; }
  .check-anim { animation: checkPop 0.3s cubic-bezier(0.34, 1.56, 0.64, 1); display: flex; }
  @keyframes checkPop { 0% { transform: scale(0); opacity: 0; } 60% { transform: scale(1.15); opacity: 1; } 100% { transform: scale(1); } }
  .address-text {
    font-family: 'Nunito Sans', sans-serif; font-weight: 500; font-size: 13px;
    color: #111827; line-height: 1.5; flex: 1;
  }
  .address-divider { height: 1px; background: #F3F4F6; margin: 0 16px; }

  .dashed-divider {
    border: none;
    border-top: 1px dashed #D1D5DB;
    margin: 12px 0;
  }

  .add-address-btn {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 12px 16px;
    width: 100%;
    text-align: center;
  }

  .add-address-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    color: #111827;
  }
</style>
