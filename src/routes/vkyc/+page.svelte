<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import { onMount, onDestroy } from 'svelte';
  import StatusBar from '$lib/components/StatusBar.svelte';
  import BottomSheet from '$lib/components/BottomSheet.svelte';
  import ShimmerText from '$lib/effects/ShimmerText.svelte';

  // ── Countdown timer ──────────────────────────────────────────
  // Starts at 42 minutes 7 seconds
  let totalSeconds = $state(42 * 60 + 7);
  let timerInterval: ReturnType<typeof setInterval>;

  let hours   = $derived(Math.floor(totalSeconds / 3600));
  let minutes = $derived(Math.floor((totalSeconds % 3600) / 60));
  let seconds = $derived(totalSeconds % 60);

  let hh = $derived(String(hours).padStart(2, '0'));
  let mm = $derived(String(minutes).padStart(2, '0'));
  let ss = $derived(String(seconds).padStart(2, '0'));

  // ── Chip pop-in + shimmer ─────────────────────────────────────
  let showChip = $state(false);
  let shimmerChip = $state(false);

  onMount(() => {
    timerInterval = setInterval(() => {
      if (totalSeconds > 0) totalSeconds -= 1;
      else clearInterval(timerInterval);
    }, 1000);
    setTimeout(() => { showChip = true; }, 1200);
    setTimeout(() => { shimmerChip = true; }, 1900);
  });

  onDestroy(() => clearInterval(timerInterval));

  // ── FAQ ─────────────────────────────────────────────
  let faqOpen = $state(false);

  let loading = $state(false);

  async function handleGetStarted() {
    loading = true;
    await new Promise(r => setTimeout(r, 500));
    loading = false;
    goto(`${base}/card-activation`);
  }
</script>

<svelte:head><title>Complete video KYC – DCB</title></svelte:head>

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
    <div class="progress-row">
      <h1 class="screen-title">Complete video KYC</h1>
    </div>
  </div>

  <!-- ── CONTENT ── -->
  <div class="content">

    <!-- Main card -->
    <div class="main-card">

      <!-- Blue highlight row: illustration + step text + timer -->
      <div class="highlight-row">

        <!-- VKYC illustration -->
        <img src="/vkyc.svg" alt="" class="vkyc-illustration" draggable="false" aria-hidden="true" />

        <!-- Right side: text + timer -->
        <div class="highlight-right">
          <div class="step-text-wrap">
            <p class="step-text">
              <span class="step-pill">Just <span class="step-num">1 Step</span> left</span>
            </p>
            <p class="step-sub">Complete it before the specified time</p>
          </div>

          <!-- Timer + info -->
          <div class="timer-row">
            <div class="timer">
              <div class="timer-block">{hh[0]}</div>
              <div class="timer-block">{hh[1]}</div>
              <span class="timer-colon">:</span>
              <div class="timer-block">{mm[0]}</div>
              <div class="timer-block">{mm[1]}</div>
              <span class="timer-colon">:</span>
              <div class="timer-block">{ss[0]}</div>
              <div class="timer-block">{ss[1]}</div>
            </div>
            <i class="ph ph-info" style="font-size:15px; color:#6B7280"></i>
          </div>
        </div>

      </div>

      <!-- Sub-heading with divider lines -->
      <div class="sub-heading-row">
        <div class="sub-line sub-line-left"></div>
        <span class="sub-heading">Keep the below things ready. Our agent will guide you</span>
        <div class="sub-line sub-line-right"></div>
      </div>

      <!-- Requirements list -->
      <div class="req-list">

        <!-- PAN card -->
        <div class="req-item req-item-blue">
          <div class="req-left">
            <div class="pan-mini">
              <img src="/pan-illustration.svg" alt="PAN card" width="42" height="28" draggable="false" />
            </div>
            <div class="req-text-col">
              <p class="req-text">Your original physical PAN card</p>
              <div class="req-badge">
                <span class="req-badge-text">This is mandatory</span>
                <i class="ph ph-info" style="font-size:16px; color:#6B7280"></i>
              </div>
            </div>
          </div>
        </div>

        <div class="req-divider"></div>

        <!-- WiFi + Light row -->
        <div class="req-row">

          <!-- WiFi -->
          <div class="req-item-sm">
            <div class="req-icon-wrap">
              <img src="/vkyc-wifi.svg" alt="WiFi" width="32" height="32" draggable="false" />
            </div>
            <p class="req-text-sm">A stable internet connection</p>
          </div>

          <div class="req-vdivider"></div>

          <!-- Light -->
          <div class="req-item-sm">
            <div class="req-icon-wrap">
              <img src="/vkyc-light.svg" alt="Light" width="32" height="32" draggable="false" />
            </div>
            <p class="req-text-sm">You alone in a well lighted space</p>
          </div>

        </div>

      </div>

    </div>

    <!-- FAQ item -->
    <button class="faq-item" onclick={() => faqOpen = true}>
      <p class="faq-text">What happens to my money if I don't complete Video KYC?</p>
      <i class="ph ph-caret-down" style="font-size:24px; color:#6B7280"></i>
    </button>

  </div>

  <div class="spacer"></div>

  <!-- ── FOOTER ── -->
  <div class="footer">
    <!-- Green chip with pop-in + shimmer -->
    {#if showChip}
      <ShimmerText trigger={shimmerChip} loop={true} duration={1600}>
        <div class="time-chip chip-pop">
          <i class="ph ph-clock" style="font-size:16px; color:#15803D"></i>
          <span class="chip-text">Takes less than <strong>5 minutes</strong></span>
        </div>
      </ShimmerText>
    {/if}

    <button class="btn-primary" onclick={handleGetStarted} disabled={loading}>
      {#if loading}
        <span class="spinner"></span>
      {:else}
        Get started
      {/if}
    </button>
  </div>

</div>

<!-- ── FAQ BOTTOM SHEET ── -->
<BottomSheet bind:open={faqOpen} title="What happens to my money if I don't complete Video KYC?">
  <div class="faq-sheet-content">
    <p>Your FD amount will be safely refunded to your source account within 5–7 working days if you don't complete Video KYC within the stipulated time.</p>
    <p>The credit card application will be cancelled and you will need to re-apply if you wish to proceed later.</p>
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
    padding: 40px 16px 0 16px; gap: 16px;
  }
  .screen-title {
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 24px;
    line-height: 1.2; color: #111827; letter-spacing: -0.25px; flex: 1;
  }

  /* ── Content ── */
  .content {
    display: flex;
    flex-direction: column;
    gap: 16px;
    padding: 20px 16px 0;
    flex-shrink: 0;
  }

  /* ── Main card ── */
  .main-card {
    display: flex;
    flex-direction: column;
    gap: 16px;
    padding: 0 12px 16px;
    border: 0.5px solid #D1D5DB;
    border-radius: 12px;
    background: #FFFCF4;
  }

  /* ── Highlight row ── */
  .highlight-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 8px;
    background: linear-gradient(180deg, #DCECFD 0%, #FFFFFF 100%);
    border-radius: 12px 12px 0 0;
    padding: 12px;
    margin: 0 -12px;
    margin-top: 0;
  }

  .vkyc-illustration {
    width: 102px;
    height: auto;
    flex-shrink: 0;
    user-select: none;
    pointer-events: none;
  }

  .highlight-right {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    gap: 12px;
    flex-shrink: 0;
  }

  .step-text-wrap {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    gap: 4px;
  }

  .step-text {
    display: flex;
  }

  .step-pill {
    display: inline-flex;
    align-items: center;
    gap: 3px;
    padding: 3px 12px;
    background: linear-gradient(90deg, #CC2B5E 0%, #753A88 100%);
    border-radius: 8px;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 12px;
    line-height: 130%;
    color: #FFFFFF;
  }

  .step-num {
    font-size: 14px;
    font-weight: 600;
    color: #FFFFFF;
    line-height: 130%;
  }

  .step-sub {
    font-family: 'Nunito Sans', sans-serif;
    font-size: 10px;
    font-weight: 400;
    line-height: 16px;
    color: #111827;
  }

  /* ── Timer ── */
  .timer-row {
    display: flex;
    align-items: center;
    gap: 4px;
  }

  .timer {
    display: flex;
    align-items: center;
    gap: 1px;
  }

  .timer-block {
    width: 17px;
    height: 24px;
    background: #FFFCF4;
    border-radius: 2px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 700;
    font-size: 12px;
    color: #111827;
    box-shadow: 0px 0.375px 0.375px rgba(0,0,0,0.25);
  }

  .timer-colon {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 700;
    font-size: 12px;
    color: #111827;
    padding: 0 1px;
    line-height: 24px;
  }

  /* ── Sub-heading with divider lines ── */
  .sub-heading-row {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .sub-line {
    flex: 1;
    height: 0;
    border: none;
    border-top: 0.5px solid transparent;
  }
  .sub-line-left {
    border-image: linear-gradient(90deg, rgba(22,23,26,0) 0%, rgba(163,171,187,1) 100%) 1;
  }
  .sub-line-right {
    border-image: linear-gradient(90deg, rgba(163,171,187,1) 0%, rgba(22,23,26,0) 100%) 1;
  }

  .sub-heading {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 10px;
    line-height: 16px;
    color: #111827;
    text-align: center;
    flex-shrink: 0;
    max-width: 200px;
  }

  /* ── Requirements list ── */
  .req-list {
    display: flex;
    flex-direction: column;
    border-radius: 12px;
    overflow: hidden;
  }

  /* PAN item */
  .req-item-blue {
    background: transparent;
    padding: 0 12px 12px;
  }

  .req-left {
    display: flex;
    align-items: center;
    gap: 12px;
  }

  .pan-mini {
    flex-shrink: 0;
  }
  .pan-mini img {
    display: block;
    user-select: none;
    pointer-events: none;
  }

  .req-text-col {
    display: flex;
    flex-direction: column;
    gap: 8px;
    flex: 1;
  }

  .req-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 12px;
    line-height: 18px;
    color: #111827;
  }

  .req-badge {
    display: flex;
    align-items: center;
    gap: 4px;
    background: #F3F4F6;
    border-radius: 500px;
    padding: 4px 8px;
    width: fit-content;
  }

  .req-badge-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 10px;
    line-height: 16px;
    color: #111827;
  }

  .req-divider {
    height: 0.5px;
    background: #D1D5DB;
  }

  /* WiFi + Light row */
  .req-row {
    display: flex;
    align-items: stretch;
    background: transparent;
  }

  .req-item-sm {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
    padding: 12px;
  }

  .req-icon-wrap {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 32px;
    height: 32px;
  }

  .req-text-sm {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 12px;
    line-height: 18px;
    color: #111827;
  }

  .req-vdivider {
    width: 0.5px;
    background: #D1D5DB;
    align-self: stretch;
  }

  /* ── FAQ ── */
  .faq-item {
    display: flex;
    align-items: center;
    gap: 16px;
    padding: 10px 12px;
    width: 100%;
    background: #FFFCF4;
    border: 1px solid #D1D5DB;
    border-radius: 8px;
    cursor: pointer;
    text-align: left;
    transition: background 0.15s;
  }
  .faq-item:active { background: #FFFCF4; }

  .faq-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 12px;
    line-height: 18px;
    color: #111827;
    flex: 1;
  }

  .faq-answer {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 18px;
    color: #6B7280;
  }

  .faq-sheet-content {
    display: flex;
    flex-direction: column;
    gap: 12px;
    padding: 12px;
    background: #FFFCF4;
    border-radius: 12px;
  }

  .faq-sheet-content p {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 14px;
    line-height: 20px;
    color: #6B7280;
  }

  /* ── Spacer ── */
  .spacer { flex: 1; }

  /* ── Footer ── */
  .footer { position: sticky; bottom: 0; z-index: 5; margin-top: auto;
    padding: 20px 16px;
    background: #FFFCF4;
    flex-shrink: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 32px;
  }

  /* Chip pop-in animation — uniform scale pop like iOS badge bounce */
  .chip-pop {
    animation: chipPop 0.6s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
    transform-origin: center center;
  }
  @keyframes chipPop {
    0%   { transform: scale(0); opacity: 0; }
    50%  { transform: scale(1.08); opacity: 1; }
    70%  { transform: scale(0.96); opacity: 1; }
    100% { transform: scale(1); opacity: 1; }
  }

  .time-chip {
    display: flex;
    align-items: center;
    gap: 4px;
    background: #F0FDF4;
    border: 0.5px solid #15803D;
    border-radius: 8px;
    padding: 4px 8px;
  }

  .chip-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 10px;
    line-height: 16px;
    color: #15803D;
  }

  .chip-text strong {
    font-weight: 600;
    font-size: 12px;
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
