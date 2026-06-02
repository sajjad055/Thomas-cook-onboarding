<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import { fly } from 'svelte/transition';
  import { cubicOut } from 'svelte/easing';
  import StatusBar from '$lib/components/StatusBar.svelte';
  import ProgressComponent from '$lib/components/ProgressComponent.svelte';
  import BottomSheet from '$lib/components/BottomSheet.svelte';

  interface Address {
    id: number;
    text: string;
  }

  let addresses = $state<Address[]>([
    { id: 1, text: 'Tanvi residency, Anugraha layout, Bilekahalli, Bangalore, Karnataka – 560076' },
  ]);

  let selectedId = $state(1);
  let understood = $state(false);
  let loading = $state(false);
  let addedSecond = $state(false);

  // Nominee sheet
  let showNomineeSheet = $state(false);
  let nomineeChoice = $state<'yes' | 'no' | null>(null);

  const secondAddress = 'G6, Perody Palace, Kodichiknahalli, Bommanahalli, Bangalore – 560076';

  function addNewAddress() {
    if (addedSecond) return;
    addedSecond = true;
    addresses = [...addresses, { id: 2, text: secondAddress }];
  }

  // Continue on address screen → open nominee sheet
  function handleContinue() {
    if (!understood) return;
    showNomineeSheet = true;
  }

  // Continue inside nominee sheet → navigate
  async function handleNomineeContinue() {
    if (!nomineeChoice) return;
    showNomineeSheet = false;
    loading = true;
    await new Promise(r => setTimeout(r, 400));
    loading = false;
    goto(`${base}/vkyc`);
  }
</script>

<svelte:head><title>Communication address – DCB</title></svelte:head>

<div class="screen">

  <!-- ── HEADER ── -->
  <div class="header-area">
    <StatusBar />
    <div class="icon-row">
      <button class="icon-btn" aria-label="Back" onclick={() => goto(`${base}/income`)}>
        <i class="ph ph-caret-left" style="font-size:24px; color:#111827"></i>
      </button>
      <div class="icon-group">
        <button class="icon-btn" aria-label="Help">
          <i class="ph-duotone ph-headset" style="font-size:24px; color:#111827"></i>
        </button>
      </div>
    </div>
    <div class="progress-row">
      <h1 class="screen-title">Select Your communication address</h1>
      <ProgressComponent step={8} totalSteps={9} />
    </div>
  </div>

  <!-- ── FORM ── -->
  <div class="form-area">

    <!-- Add new address -->
    <button class="add-row" onclick={addNewAddress} disabled={addedSecond}>
      <i class="ph ph-plus" style="font-size:24px; color:#242A80"></i>
      <span class="add-text">Add new address</span>
    </button>

    <!-- Address list -->
    <div class="address-list">
      {#each addresses as addr (addr.id)}
        <button
          class="address-card"
          class:selected={selectedId === addr.id}
          onclick={() => selectedId = addr.id}
          in:fly={{ y: 16, duration: 280, easing: cubicOut }}
        >
          <!-- Radio -->
          <div class="radio" class:checked={selectedId === addr.id}>
            {#if selectedId === addr.id}
              <div class="check-anim">
                <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                  <circle cx="10" cy="10" r="10" fill="#15803D"/>
                  <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </div>
            {/if}
          </div>

          <!-- Address text -->
          <p class="address-text">{addr.text}</p>
        </button>
      {/each}
    </div>

  </div>

  <div class="spacer"></div>

  <!-- ── FOOTER ── -->
  <div class="footer">

    <!-- Checkbox row -->
    <div class="checkbox-row">
      <button
        class="checkbox"
        class:checked={understood}
        onclick={() => understood = !understood}
        aria-label="I understand"
        role="checkbox"
        aria-checked={understood}
      >
        {#if understood}
          <div class="check-anim">
            <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
              <rect width="20" height="20" rx="4" fill="#242A80"/>
              <path d="M5.5 10.2L8.2 13L14.5 6.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </div>
        {:else}
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
            <rect x="0.5" y="0.5" width="19" height="19" rx="3.5" stroke="#D1D5DB"/>
          </svg>
        {/if}
      </button>
      <p class="checkbox-text">
        I understand that the card will be delivered to the above selected address
      </p>
    </div>

    <!-- CTA -->
    <button
      class="btn-primary"
      disabled={!understood || loading}
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

  /* ── Add new address ── */
  .add-row {
    display: flex;
    align-items: center;
    gap: 8px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;
    transition: opacity 0.15s;
  }
  .add-row:disabled { opacity: 0.4; cursor: not-allowed; }

  .add-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    line-height: 20px;
    color: #242A80;
  }

  /* ── Address list ── */
  .address-list {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  /* ── Address card ── */
  .address-card {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 12px;
    width: 100%;
    background: #FFFDF6;
    border: 1px solid #D1D5DB;
    border-radius: 12px;
    cursor: pointer;
    text-align: left;
  }
  .address-card:active { opacity: 0.9; }

  /* ── Radio ── */
  .radio {
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

  .address-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    line-height: 20px;
    color: #111827;
    flex: 1;
  }

  /* ── Spacer ── */
  .spacer { flex: 1; }

  /* ── Footer ── */
  .footer { position: sticky; bottom: 0; z-index: 5; margin-top: auto;
    padding: 16px;
    background: #FFFDF6;
    flex-shrink: 0;
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  /* ── Checkbox row ── */
  .checkbox-row {
    display: flex;
    align-items: flex-start;
    gap: 4px;
  }

  .checkbox {
    width: 24px;
    height: 24px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 2px;
    flex-shrink: 0;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .checkbox-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 18px;
    color: #6B7280;
    flex: 1;
  }

  /* ── CTA ── */
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

  /* ── Nominee sheet list ── */
  .nw-list {
    display: flex;
    flex-direction: column;
    background: #FFFDF6;
    border-radius: 12px;
    overflow: hidden;
    border: 1px solid #F3F4F6;
  }

  .nw-item {
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
  .nw-item:active { background: #FFFDF6; }

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
    background: #FFFDF6;
  }
  .nw-radio.checked { border: none; background: none; }

  .nw-divider {
    height: 1px;
    background: #F3F4F6;
    margin: 0 12px;
  }
</style>

<!-- ── NOMINEE BOTTOM SHEET ── -->
<BottomSheet bind:open={showNomineeSheet} title="Do you want to add a nominee for your FD?">

  <div class="nw-list">

    <button class="nw-item" onclick={() => nomineeChoice = 'yes'}>
      <span class="nw-text">Yes, I want to add a nominee</span>
      <div class="nw-radio" class:checked={nomineeChoice === 'yes'}>
        {#if nomineeChoice === 'yes'}
          <div class="check-anim">
            <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
              <circle cx="10" cy="10" r="10" fill="#15803D"/>
              <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </div>
        {/if}
      </div>
    </button>

    <div class="nw-divider"></div>

    <button class="nw-item" onclick={() => nomineeChoice = 'no'}>
      <span class="nw-text">No, I'll continue without a nominee</span>
      <div class="nw-radio" class:checked={nomineeChoice === 'no'}>
        {#if nomineeChoice === 'no'}
          <div class="check-anim">
            <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
              <circle cx="10" cy="10" r="10" fill="#15803D"/>
              <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </div>
        {/if}
      </div>
    </button>

  </div>

  {#snippet footer()}
    <button
      class="btn-primary"
      disabled={!nomineeChoice || loading}
      onclick={handleNomineeContinue}
    >
      {#if loading}<span class="spinner"></span>{:else}Continue{/if}
    </button>
  {/snippet}

</BottomSheet>
