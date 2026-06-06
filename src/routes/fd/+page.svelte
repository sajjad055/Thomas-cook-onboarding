<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import StatusBar from '$lib/components/StatusBar.svelte';

  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';

  const fdOptions = [
    { label: '₹10,000', value: 10000 },
    { label: '₹25,000', value: 25000 },
    { label: '₹50,000', value: 50000 },
  ];

  let selectedFd = $state(25000);
  let customMode = $state(false);
  let customValue = $state('');
  let loading = $state(false);

  let fdAmount = $derived(customMode ? (parseInt(customValue) || 0) : selectedFd);
  let creditLimit = $derived(Math.round(fdAmount * 0.9));
  const animatedCredit = tweened(22500, { duration: 1200, easing: cubicOut });
  $effect(() => { animatedCredit.set(creditLimit); });
  let formattedCredit = $derived('₹' + Math.round($animatedCredit).toLocaleString('en-IN'));
  let formattedFd = $derived('₹' + fdAmount.toLocaleString('en-IN'));

  function selectFd(val: number) {
    customMode = false;
    selectedFd = val;
  }

  function enterCustom() {
    customMode = true;
    customValue = '';
  }

  async function handleProceed() {
    if (fdAmount < 10000) return;
    loading = true;
    await new Promise(r => setTimeout(r, 500));
    loading = false;
    goto(`${base}/payment`);
  }
</script>

<svelte:head><title>Select FD & set limit – IOB</title></svelte:head>

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
      <img src="{base}/security.svg" alt="" class="illustration-img" />
    </div>

    <div class="progress-row">
      <h1 class="screen-title">Select FD & set limit</h1>
    </div>
  </div>

  <!-- ── FD SELECTION CARD ── -->
  <div class="fd-section">
    <div class="fd-card">

      <!-- Top: credit limit display -->
      <div class="fd-top">
        <div class="fd-badge-row">
          <div class="fd-badge">
            <span class="fd-badge-text">TC travel card limit</span>
          </div>
        </div>
        <div class="fd-amount-row">
          <span class="fd-amount">{formattedCredit}</span>
        </div>
        <div class="fd-subtitle-row">
          <span class="fd-subtitle">Credit limit is 90% of your FD amount</span>
        </div>
      </div>

      <!-- Divider with pill label — Figma: row, center, gap:10px, px:12px, fill -->
      <div class="fd-divider-wrap">
        <div class="fd-divider-inner">
          <div class="fd-divider-content">
            <div class="fd-divider-row">
              <div class="fd-line fd-line-left"></div>
              <div class="fd-divider-pill">
                <span class="fd-divider-text">Select from popular FDs</span>
              </div>
              <div class="fd-line fd-line-right"></div>
            </div>
          </div>
        </div>
      </div>

      <!-- FD amount pills -->
      <div class="fd-pills-section">
        <div class="fd-pills-row">
          {#each fdOptions as opt}
            <button
              class="fd-pill"
              class:selected={!customMode && selectedFd === opt.value}
              onclick={() => selectFd(opt.value)}
            >
              {opt.label}
            </button>
          {/each}
          <button
            class="fd-pill fd-pill-custom"
            class:selected={customMode}
            onclick={enterCustom}
          >
            Custom
          </button>
        </div>

        {#if customMode}
          <div class="custom-input-row">
            <span class="custom-prefix">₹</span>
            <input
              type="tel"
              inputmode="numeric"
              placeholder="Enter amount"
              bind:value={customValue}
              class="custom-input"
              maxlength="7"
            />
          </div>
        {/if}

        <span class="fd-note">Your FD renews automatically in 365 days.</span>
      </div>

      <!-- Green footer -->
      <div class="fd-interest-row">
        <span class="fd-interest">Interest earned on FD : 7.0% P.A</span>
      </div>

    </div>
  </div>

  <!-- ── CARD BENEFITS MODULE ── -->
  <div class="benefits-section">
    <div class="benefits-card">
      <!-- Top: Banner SVG -->
      <div class="benefits-banner">
        <img src="{base}/card-benefits-banner.svg" alt="Card benefits" width="100%" draggable="false" />
      </div>

      <!-- Bottom: Offers ribbon -->
      <div class="offers-ribbon">
        <div class="offers-ribbon-bg"></div>
        <div class="offers-content">
          <div class="offers-top-row">
            <div class="offer-tag-wrap">
              <img src="{base}/offer-tag.svg" alt="" width="31" height="31" draggable="false" />
            </div>
            <div class="offers-label-col">
              <span class="offers-label">Offers on<br/>top brands</span>
            </div>
          </div>
          <div class="brand-marquee-mask">
            <div class="brand-marquee-track">
              <img src="{base}/flipkart.svg" alt="Flipkart" class="brand-logo" />
              <img src="{base}/amazon.svg" alt="Amazon" class="brand-logo" />
              <img src="{base}/myntra.svg" alt="Myntra" class="brand-logo" />
              <img src="{base}/netflix.svg" alt="Netflix" class="brand-logo" />
              <img src="{base}/swiggy.svg" alt="Swiggy" class="brand-logo" />
              <!-- Duplicate set for seamless loop -->
              <img src="{base}/flipkart.svg" alt="" class="brand-logo" aria-hidden="true" />
              <img src="{base}/amazon.svg" alt="" class="brand-logo" aria-hidden="true" />
              <img src="{base}/myntra.svg" alt="" class="brand-logo" aria-hidden="true" />
              <img src="{base}/netflix.svg" alt="" class="brand-logo" aria-hidden="true" />
              <img src="{base}/swiggy.svg" alt="" class="brand-logo" aria-hidden="true" />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Bank Partner Logos -->
  <div class="partner-logos">
    <img src="{base}/logos-bank-partner.svg" alt="Bank Partners" class="partner-img" draggable="false" />
  </div>

  <div class="spacer"></div>

  <!-- ── FOOTER ── -->
  <div class="footer">
    <div class="footer-inner">
      <div class="footer-info">
        <span class="footer-label">Payable FD amount</span>
        <span class="footer-amount">{formattedFd}</span>
      </div>
      <button
        class="btn-primary"
        disabled={fdAmount < 10000 || loading}
        onclick={handleProceed}
      >
        {#if loading}
          <span class="spinner"></span>
        {:else}
          Proceed to payment
        {/if}
      </button>
    </div>
  </div>

</div>

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
  .icon-row { display: flex; align-items: center; justify-content: space-between; width: 100%; }
  .icon-group { display: flex; align-items: center; }
  .icon-btn {
    display: flex; align-items: center; justify-content: center;
    width: 48px; height: 48px; background: none; border: none;
    cursor: pointer; border-radius: 50%; transition: background 0.15s;
  }
  .icon-btn:active { background: rgba(0,0,0,0.06); }
  .illustration-frame { width: 60px; height: 60px; border-radius: 50%; background: #F3F4F6; border: 0.5px solid #F5F5F5; margin-left: 16px; flex-shrink: 0; display: flex; align-items: center; justify-content: center; overflow: hidden; }
  .illustration-img { width: 100%; height: 100%; object-fit: cover; }
  .progress-row { display: flex; align-items: flex-end; justify-content: space-between; padding: 16px 16px 0 16px; gap: 16px; }
  .screen-title {
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 24px;
    line-height: 1.2; color: #111827; letter-spacing: -0.25px; flex: 1;
  }

  /* ── FD Card ── */
  .fd-section { padding: 0 16px; flex-shrink: 0; }

  .fd-card {
    background: #FFFCF4;
    border: 1px solid #D1D5DB;
    border-radius: 12px;
    overflow: hidden;
  }

  /* Top section */
  .fd-top {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
    padding: 12px 12px 16px;
  }

  .fd-badge-row { display: flex; justify-content: center; }
  .fd-badge {
    background: linear-gradient(154deg, #0D0D0D 0%, #353535 100%);
    border: 0.5px solid #D1D5DB;
    border-radius: 20px;
    padding: 2px 8px;
    box-shadow: inset 0px 2px 4px rgba(0,0,0,0.25);
  }
  .fd-badge-text {
    font-family: 'Nunito Sans', sans-serif; font-weight: 400; font-size: 10px;
    line-height: 1.6; color: #FFFFFF; text-align: center; display: block;
  }

  .fd-amount-row { display: flex; justify-content: center; align-items: center; }
  .fd-amount {
    font-family: 'Nunito Sans', sans-serif; font-weight: 700; font-size: 24px;
    line-height: 1.32; letter-spacing: -0.04em; color: #111827;
  }

  .fd-subtitle-row { display: flex; justify-content: center; }
  .fd-subtitle {
    font-family: 'Nunito Sans', sans-serif; font-weight: 400; font-size: 10px;
    line-height: 1.6; color: #6B7280;
  }

  /* Divider with pill — matches Figma node 17:5056 */
  .fd-divider-wrap {
    display: flex;
    align-items: center;
    gap: 24px;
    padding: 0 12px;
    align-self: stretch;
  }
  .fd-divider-inner {
    display: flex;
    gap: 8px;
    flex: 1;
  }
  .fd-divider-content {
    display: flex;
    flex-direction: column;
    justify-content: center;
    gap: 4px;
    flex: 1;
  }
  .fd-divider-row {
    display: flex;
    justify-content: center;
    align-items: center;
    align-self: stretch;
    gap: 10px;
  }
  .fd-line {
    flex: 1;
    height: 0;
    border: none;
    border-top: 0.5px solid transparent;
  }
  .fd-line-left {
    border-image: linear-gradient(90deg, rgba(22,23,26,0) 0%, rgba(163,171,187,1) 100%) 1;
  }
  .fd-line-right {
    border-image: linear-gradient(90deg, rgba(163,171,187,1) 0%, rgba(22,23,26,0) 100%) 1;
  }
  .fd-divider-pill {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 10px;
    padding: 2px 8px;
    background: linear-gradient(90deg, #AA076B 0%, #610460 100%);
    border-radius: 32px;
    flex-shrink: 0;
  }
  .fd-divider-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 10px;
    line-height: 1.6;
    color: #FFFFFF;
    text-align: center;
    white-space: nowrap;
  }

  /* Pills */
  .fd-pills-section {
    display: flex; flex-direction: column; align-items: center; gap: 8px; padding: 16px 12px;
  }

  .fd-pills-row {
    display: flex; align-items: center; gap: 8px; width: 100%;
  }

  .fd-pill {
    flex: 1;
    display: flex; justify-content: center; align-items: center;
    padding: 8px 12px;
    border: 1px solid #D1D5DB;
    border-radius: 8px;
    background: #FFFCF4;
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 14px;
    color: #111827; cursor: pointer;
    transition: all 0.15s ease;
  }
  .fd-pill.selected {
    background: #ECF2FF;
    border-color: #242A80;
    color: #242A80;
  }
  .fd-pill:active { opacity: 0.85; }

  .fd-pill-custom {
    font-size: 12px;
  }

  .custom-input-row {
    display: flex; align-items: center; gap: 4px;
    background: #FFFCF4; border: 1px solid #242A80;
    border-radius: 8px; padding: 8px 12px; width: calc(100% - 56px);
  }
  .custom-prefix {
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 14px; color: #111827;
  }
  .custom-input {
    flex: 1; background: none; border: none; outline: none;
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 14px; color: #111827;
  }
  .custom-input::placeholder { color: #9CA3AF; font-weight: 400; }

  .fd-note {
    font-family: 'Nunito Sans', sans-serif; font-weight: 400; font-size: 10px;
    line-height: 1.6; color: #6B7280;
  }

  /* Green interest row */
  .fd-interest-row {
    display: flex; justify-content: center; align-items: center;
    padding: 12px 0; gap: 3px;
    border-top: 0.5px solid #D1D5DB;
  }
  .fd-interest {
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 12px;
    line-height: 1.5; color: #15803D; text-align: center;
  }

  /* ── Spacer ── */
  .spacer { flex: 1; }

  /* ══════════════════════════════════════
     CARD BENEFITS MODULE
  ══════════════════════════════════════ */
  .benefits-section {
    padding: 0 16px;
    flex-shrink: 0;
    margin-top: 20px;
  }

  .benefits-card {
    border: 1px solid #D1D5DB;
    border-radius: 16px;
    overflow: visible;
    display: flex;
    flex-direction: column;
  }

  .benefits-banner {
    width: 100%;
    overflow: hidden;
    border-radius: 16px 16px 0 0;
  }
  .benefits-banner img {
    display: block;
    width: 100%;
    height: 100%;
    object-fit: cover;
    user-select: none;
    pointer-events: none;
  }

  /* Offers ribbon */
  .offers-ribbon {
    position: relative;
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 10px 16px 14px;
    overflow: visible;
    background: linear-gradient(90deg, #EAEAEA 0%, #FFFFFF 100%);
  }

  .offers-ribbon-bg {
    position: absolute;
    inset: 0;
    background: linear-gradient(82deg, rgba(255,246,204,1) 61%, rgba(255,255,255,0) 100%);
    filter: blur(50px);
    pointer-events: none;
  }

  .offers-content {
    position: relative;
    display: flex;
    align-items: center;
    gap: 4px;
    width: 100%;
  }

  .offers-top-row {
    display: flex;
    align-items: center;
    gap: 4px;
    flex-shrink: 0;
  }

  .offer-tag-wrap {
    position: relative;
    width: 31px;
    height: 31px;
    flex-shrink: 0;
  }
  .offer-tag-wrap img {
    display: block;
    width: 100%;
    height: 100%;
  }

  .offers-label-col {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
  }

  .offers-label {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 10px;
    line-height: 1.4;
    color: #111827;
    white-space: nowrap;
  }

  /* Marquee mask — clips the scrolling logos, fades edges */
  .brand-marquee-mask {
    flex: 1;
    overflow: hidden;
    position: relative;
    mask-image: linear-gradient(90deg, transparent 0%, black 10%, black 90%, transparent 100%);
    -webkit-mask-image: linear-gradient(90deg, transparent 0%, black 10%, black 90%, transparent 100%);
  }

  /* Scrolling track — holds 2× logos for seamless loop */
  .brand-marquee-track {
    display: flex;
    align-items: center;
    gap: 20px;
    width: max-content;
    animation: marqueeScroll 8s linear infinite;
  }

  @keyframes marqueeScroll {
    0%   { transform: translateX(0); }
    100% { transform: translateX(-50%); }
  }

  .brand-logo {
    width: 22px;
    height: 22px;
    border-radius: 5px;
    box-shadow: 0px 1px 4px rgba(0,0,0,0.25);
    object-fit: contain;
    user-select: none;
    pointer-events: none;
    flex-shrink: 0;
  }

  /* ── Footer ── */
  .footer { position: sticky; bottom: 0; z-index: 5; margin-top: auto;
    background: #FFFCF4;
    flex-shrink: 0;
  }
  .footer-inner {
    display: flex; align-items: flex-end; gap: 16px; padding: 16px;
  }
  .footer-info {
    display: flex; flex-direction: column; gap: 4px; flex-shrink: 0;
  }
  .footer-label {
    font-family: 'Nunito Sans', sans-serif; font-weight: 400; font-size: 12px;
    line-height: 1.5; color: #6B7280;
  }
  .footer-amount {
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 16px;
    line-height: 1.375; color: #111827;
  }

  .btn-primary {
    display: flex; align-items: center; justify-content: center;
    flex: 1; height: 48px; background: #184595; color: #FFFFFF;
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 16px;
    border: none; border-radius: 8px; cursor: pointer;
    transition: opacity 0.15s, transform 0.1s;
  }
  .btn-primary:active:not(:disabled) { opacity: 0.88; transform: scale(0.99); }
  .btn-primary:disabled { background: #D1D5DB; cursor: not-allowed; }

  .spinner {
    width: 20px; height: 20px;
    border: 2.5px solid rgba(255,255,255,0.4); border-top-color: #FFFFFF;
    border-radius: 50%; animation: spin 0.7s linear infinite;
  }
  @keyframes spin { to { transform: rotate(360deg); } }

  /* Partner logos */
  .partner-logos {
    display: flex;
    justify-content: center;
    margin-top: 40px;
    flex-shrink: 0;
  }
  .partner-img {
    height: 38px;
    filter: grayscale(100%);
    width: auto;
    user-select: none;
    pointer-events: none;
  }
</style>
