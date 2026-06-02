<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import StatusBar from '$lib/components/StatusBar.svelte';
  import ProgressComponent from '$lib/components/ProgressComponent.svelte';

  import { onMount, onDestroy } from 'svelte';

  let loading = $state(false);
  let proofIndex = $state(0);

  const proofs = [
    { type: 'icon', icon: 'ph-users', text: '500 customers booked FDs in the last 5 days.' },
    { type: 'img', src: '/upi.svg', text: '90% of payments are completed via UPI instantly.' },
  ];

  let proofInterval: ReturnType<typeof setInterval>;
  onMount(() => {
    proofInterval = setInterval(() => {
      proofIndex = (proofIndex + 1) % proofs.length;
    }, 3500);
  });
  onDestroy(() => clearInterval(proofInterval));

  function handleUpiPay(app: string) {
    goto(`${base}/payment/success`);
  }

  function handleAddBank() {
    goto(`${base}/payment/add-bank`);
  }
</script>

<svelte:head><title>Make a payment – DCB</title></svelte:head>

<div class="screen">

  <!-- ── HEADER ── -->
  <div class="header-area">
    <StatusBar />
    <div class="icon-row">
      <button class="icon-btn" aria-label="Back" onclick={() => goto(`${base}/fd`)}>
        <i class="ph ph-caret-left"></i>
      </button>
      <div class="icon-group">
        <button class="icon-btn" aria-label="Help">
          <i class="ph-duotone ph-headset" style="font-size:24px"></i>
        </button>
      </div>
    </div>
    <div class="progress-row">
      <div class="title-col">
        <h1 class="screen-title">Make a payment</h1>
      </div>
      <ProgressComponent step={5} totalSteps={9} />
    </div>
  </div>

  <!-- ── FORM AREA ── -->
  <div class="form-area">

    <!-- Payable amount card -->
    <div class="amount-card">
      <span class="amount-label">Payable amount</span>
      <span class="amount-value">₹25,000</span>
    </div>

    <!-- Pay with UPI apps -->
    <div class="section">
      <div class="section-header">
        <span class="section-title">Pay with UPI apps</span>
        <div class="success-chip">
          <img src="/bolt-green.svg" width="18" height="18" alt="" aria-hidden="true" />
          <span class="chip-text">High payment success rate</span>
        </div>
      </div>

      <div class="list-card">
        <!-- Google Pay -->
        <!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
        <div class="list-item" onclick={() => handleUpiPay('Google Pay')}>
          <div class="list-icon gpay">
            <img src="/googlepay.svg" alt="Google Pay" width="27" height="27" />
          </div>
          <div class="list-text">
            <span class="list-title">Google pay</span>
          </div>
          <i class="ph ph-caret-right" style="font-size:20px; color:#6B7280"></i>
        </div>

        <div class="list-divider"></div>

        <!-- PhonePe -->
        <!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
        <div class="list-item" onclick={() => handleUpiPay('PhonePe')}>
          <div class="list-icon phonepe">
            <img src="/phonepe.svg" alt="PhonePe" width="27" height="27" />
          </div>
          <div class="list-text">
            <span class="list-title">Phonepe</span>
          </div>
          <i class="ph ph-caret-right" style="font-size:20px; color:#6B7280"></i>
        </div>
      </div>
    </div>

    <!-- Pay via bank account -->
    <div class="section">
      <div class="section-header">
        <span class="section-title">Pay via bank account</span>
      </div>

      <!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
      <div class="list-card-single" onclick={handleAddBank}>
        <div class="list-item">
          <div class="list-icon bank-icon">
            <i class="ph ph-bank" style="font-size:20px; color:#6B7280"></i>
          </div>
          <div class="list-text">
            <span class="list-title">Add bank account</span>
          </div>
          <i class="ph ph-caret-right" style="font-size:20px; color:#6B7280"></i>
        </div>
      </div>
    </div>

  </div>

  <!-- Social proof — rotating messages -->
  <div class="social-proof">
    {#key proofIndex}
      <div class="proof-item">
        {#if proofs[proofIndex].type === 'icon'}
          <i class="ph {proofs[proofIndex].icon}" style="font-size:20px;color:#111827"></i>
        {:else}
          <img src={proofs[proofIndex].src} alt="" width="28" height="28" style="flex-shrink:0" />
        {/if}
        <span class="social-text">{proofs[proofIndex].text}</span>
      </div>
    {/key}
  </div>

  <div class="spacer"></div>

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
  .progress-row { display: flex; align-items: flex-end; justify-content: space-between; padding: 0 16px; gap: 16px; }
  .title-col { display: flex; flex-direction: column; gap: 3px; flex: 1; }
  .screen-title {
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 24px;
    line-height: 1.2; color: #111827; letter-spacing: -0.25px;
  }

  /* ── Form area ── */
  .form-area {
    padding: 20px 0 0;
    border-radius: 16px 16px 0 0;
    display: flex;
    flex-direction: column;
    gap: 24px;
    flex-shrink: 0;
  }

  /* Amount card */
  .amount-card {
    display: flex;
    flex-direction: column;
    gap: 4px;
    padding: 0 16px;
  }
  .amount-label {
    font-family: 'Nunito Sans', sans-serif; font-weight: 400; font-size: 12px;
    line-height: 1.5; color: #6B7280;
  }
  .amount-value {
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 24px;
    line-height: 1.32; color: #111827;
  }

  /* Sections */
  .section {
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  .section-header {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 0 16px;
  }

  .section-title {
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 12px;
    line-height: 1.5; color: #6B7280;
  }

  .success-chip {
    display: flex;
    align-items: center;
    gap: 4px;
    padding: 2px 8px;
    background: #F0FDF4;
    border: 0.25px solid #15803D;
    border-radius: 32px;
  }
  .chip-text {
    font-family: 'Nunito Sans', sans-serif; font-weight: 400; font-size: 10px;
    line-height: 1.6; color: #111827; text-align: center;
  }

  /* List cards */
  .list-card {
    display: flex;
    flex-direction: column;
    margin: 0 16px;
    border: 0.5px solid #D1D5DB;
    border-radius: 12px;
    overflow: hidden;
  }

  .list-card-single {
    margin: 0 16px;
    border: 0.5px solid #D1D5DB;
    border-radius: 12px;
    overflow: hidden;
    cursor: pointer;
  }
  .list-card-single:active { opacity: 0.9; }

  .list-item {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 12px;
    background: #FFFDF6;
    cursor: pointer;
    transition: background 0.1s;
  }
  .list-item:active { background: #FFFDF6; }

  .list-icon {
    width: 36px;
    height: 36px;
    border-radius: 36px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    border: 0.45px solid #E1E6EC;
    overflow: hidden;
  }
  .list-icon img {
    width: 100%;
    height: 100%;
    object-fit: contain;
  }
  .list-icon.gpay { background: #FFFDF6; }
  .list-icon.phonepe { background: #F3EEFA; }
  .list-icon.bank-icon { background: #FFFDF6; border-radius: 100px; }

  .list-text {
    display: flex;
    flex-direction: column;
    gap: 4px;
    flex: 1;
  }
  .list-title {
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 14px;
    line-height: 1.43; color: #111827;
  }

  .list-divider {
    height: 0;
    border-top: 0.5px dashed #D1D5DB;
    margin: 0 12px;
  }

  /* Social proof */
  .social-proof {
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 8px 16px;
    height: 36px;
    background: linear-gradient(0deg, #D3B1FF 0%, #F5EDFF 31%, #FCF9FF 100%);
    flex-shrink: 0;
    margin-top: 32px;
    overflow: hidden;
    position: relative;
  }
  .proof-item {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    animation: proofFade 3.5s ease;
  }
  @keyframes proofFade {
    0%   { opacity: 0; transform: translateY(8px); }
    10%  { opacity: 1; transform: translateY(0); }
    85%  { opacity: 1; transform: translateY(0); }
    100% { opacity: 0; transform: translateY(-8px); }
  }
  .social-text {
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 12px;
    line-height: 1.5; color: #111827;
  }

  .spacer { flex: 1; }
</style>
