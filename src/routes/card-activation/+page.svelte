<script lang="ts">
  import { onMount } from 'svelte';
  import { fly, fade } from 'svelte/transition';
  import { cubicOut } from 'svelte/easing';
  import { browser } from '$app/environment';
  import StatusBar from '$lib/components/StatusBar.svelte';

  // Phase 1: Card centered → Phase 2: Card moves up, content reveals
  let phase = $state<'intro' | 'reveal'>('intro');
  let showConfetti = $state(false);

  // Staggered content reveals (phase 2)
  let showHeading = $state(false);
  let showSteps = $state(false);
  let showStep1 = $state(false);
  let showStep1Check = $state(false);
  let showStep1Line = $state(false);
  let showStep2 = $state(false);
  let showStep3 = $state(false);
  let showInfo = $state(false);

  // Generate confetti — using Lottie

  onMount(async () => {
    if (browser) {
      try { await import('@lottiefiles/dotlottie-wc'); } catch {}
    }

    // Fire confetti during intro
    setTimeout(() => showConfetti = true, 300);

    // Phase 1: hold centered for 1.5s (extra time for confetti), then transition
    setTimeout(() => {
      showConfetti = false;
      phase = 'reveal';

      // Cascade content after card moves up
      setTimeout(() => showHeading = true, 600);
      setTimeout(() => showSteps = true, 900);
      setTimeout(() => showStep1 = true, 1100);
      setTimeout(() => showStep1Check = true, 1500);
      setTimeout(() => showStep1Line = true, 1900);
      setTimeout(() => showStep2 = true, 2300);
      setTimeout(() => showStep3 = true, 2900);
      setTimeout(() => showInfo = true, 3300);
    }, 1500);
  });
</script>

<svelte:head><title>Getting your card ready – IOB</title></svelte:head>

<div class="screen">

  <!-- Blurred gradient bg ellipse -->
  <div class="bg-ellipse" aria-hidden="true"></div>

  <!-- Rotating bg lighting — moves with the card -->
  <div class="bg-card-group" class:centered={phase === 'intro'} class:settled={phase === 'reveal'}>
    <img src="/bg-lighting.svg" alt="" class="bg-lighting" draggable="false" aria-hidden="true" />
    <img src="/dcb-payless-card.svg" alt="IOB Payless Card" class="card-img" draggable="false" />
  </div>

  <!-- Confetti burst during intro -->
  {#if showConfetti}
    <div class="confetti-container" aria-hidden="true">
      <dotlottie-wc
        src="/confetti.lottie"
        autoplay
        speed="1"
        style="width: 432px; height: 480px;"
      ></dotlottie-wc>
    </div>
  {/if}

  <!-- Status bar + nav (always visible) -->
  <div class="top-bar">
    <StatusBar />
    <div class="icon-row">
      <div class="icon-spacer"></div>
      <button class="icon-btn" aria-label="Help">
        <i class="ph-duotone ph-headset" style="font-size:24px; color:#111827"></i>
      </button>
    </div>
  </div>

  <!-- Spacer to push content below the card area -->
  <div class="card-spacer"></div>

  <!-- Content area — only shows in reveal phase -->
  {#if phase === 'reveal'}

    <!-- Heading -->
    {#if showHeading}
      <p class="heading" in:fade={{ duration: 450 }}>
        Hang tight, we are getting your<br/>card ready
      </p>
    {/if}

    <!-- Steps card -->
    {#if showSteps}
      <div class="steps-area" in:fade={{ duration: 400 }}>
        <div class="steps-card">

          <div class="stepper">

            <!-- Step 1: Completed -->
            {#if showStep1}
              <div class="stepper-item" in:fade={{ duration: 400 }}>
                <div class="stepper-rail">
                  {#if showStep1Check}
                    <div class="stepper-icon stepper-icon-success check-pop-in">
                      <i class="ph ph-check" style="font-size:14px; color:#FFFFFF"></i>
                    </div>
                  {:else}
                    <div class="stepper-icon stepper-icon-pending">
                      <span class="stepper-dot"></span>
                    </div>
                  {/if}
                  {#if showStep1Line}
                    <div class="stepper-line stepper-line-success line-grow"></div>
                  {/if}
                </div>
                <div class="stepper-content">
                  <p class="step-title">Video KYC approval</p>
                  <p class="step-desc">Your verification with the bank agent</p>
                </div>
              </div>
            {/if}

            <!-- Step 2: Pending -->
            {#if showStep2}
              <div class="stepper-item" in:fade={{ duration: 500 }}>
                <div class="stepper-rail">
                  <div class="stepper-icon stepper-icon-pending">
                    <span class="stepper-dot"></span>
                  </div>
                  <div class="stepper-line stepper-line-pending"></div>
                </div>
                <div class="stepper-content">
                  <p class="step-title">Bank verification</p>
                  <p class="step-desc">Bank will verify your details and initiate FD and card creation</p>
                </div>
              </div>
            {/if}

            <!-- Step 3: Pending -->
            {#if showStep3}
              <div class="stepper-item" in:fade={{ duration: 500 }}>
                <div class="stepper-rail">
                  <div class="stepper-icon stepper-icon-pending">
                    <span class="stepper-dot"></span>
                  </div>
                </div>
                <div class="stepper-content">
                  <p class="step-title">Card setup</p>
                  <p class="step-desc">You will get a credit card and an FD of ₹25,000. <strong>You will be notified via SMS/email when your card is created</strong></p>
                </div>
              </div>
            {/if}

          </div>

          <!-- Processing badge -->
          <div class="processing-badge">
            <span class="processing-text">Processing typically requires upto 48 hours</span>
          </div>

        </div>
      </div>
    {/if}

    <!-- Info box -->
    {#if showInfo}
      <div class="info-box" in:fade={{ duration: 400 }}>
        <i class="ph ph-info" style="font-size:24px; color:#6B7280; flex-shrink:0"></i>
        <p class="info-text">
          <strong>Incase of a verification failure, your money will be refunded back to your source account within 48-72 hours</strong>
        </p>
      </div>
    {/if}

  {/if}

</div>

<style>
  .screen {
    width: 100%;
    min-height: 100dvh;
    display: flex;
    flex-direction: column;
    align-items: center;
    background: #FFFCF4;
    position: relative;
    overflow: hidden;
  }

  /* ── BG ellipse ── */
  .bg-ellipse {
    position: absolute;
    width: 639px;
    height: 639px;
    left: -139px;
    top: -93px;
    border-radius: 50%;
    background: linear-gradient(90deg, #FFEEEE 0%, #DDEFBB 100%);
    filter: blur(25.6px);
    pointer-events: none;
    z-index: 0;
  }

  /* ── Card + BG lighting group — transitions vertically from center to top ── */
  .bg-card-group {
    position: absolute;
    width: 325px;
    height: 322px;
    left: 50%;
    margin-left: -162.5px;
    top: 0;
    z-index: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    will-change: transform, opacity;
    transition: transform 1s cubic-bezier(0.25, 0.1, 0.25, 1), opacity 0.6s ease;
  }

  /* Phase 1: centered vertically */
  .bg-card-group.centered {
    transform: translateY(calc(400px - 161px));
    opacity: 1;
  }

  /* Phase 2: settled near top */
  .bg-card-group.settled {
    transform: translateY(85px);
    opacity: 1;
  }

  .bg-lighting {
    position: absolute;
    width: 325px;
    height: 322px;
    opacity: 0.79;
    pointer-events: none;
    user-select: none;
    transform-origin: center center;
    animation: bgRotate 30s linear infinite;
  }
  @keyframes bgRotate {
    from { transform: rotate(0deg); }
    to   { transform: rotate(360deg); }
  }

  .card-img {
    width: 258px;
    height: auto;
    display: block;
    user-select: none;
    pointer-events: none;
    position: relative;
    z-index: 1;
  }

  /* ── Top bar ── */
  .top-bar {
    position: relative;
    z-index: 2;
    width: 100%;
  }

  .icon-row {
    display: flex;
    align-items: center;
    justify-content: flex-end;
    width: 100%;
  }
  .icon-spacer { flex: 1; }
  .icon-btn {
    display: flex; align-items: center; justify-content: center;
    width: 48px; height: 48px; background: none; border: none;
    cursor: pointer; border-radius: 50%;
  }

  /* Spacer to account for the card area at top */
  .card-spacer {
    height: 288px;
    flex-shrink: 0;
  }

  /* ── Heading ── */
  .heading {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 20px;
    line-height: 24px;
    color: #111827;
    text-align: center;
    margin-top: 16px;
    position: relative;
    z-index: 1;
  }

  /* ── Steps ── */
  .steps-area {
    padding: 0 16px;
    margin-top: 24px;
    width: 100%;
    position: relative;
    z-index: 1;
  }

  .steps-card {
    background: #FFFCF4;
    border-radius: 8px;
    padding: 44px 12px 16px;
    display: flex;
    flex-direction: column;
    gap: 16px;
    position: relative;
    overflow: visible;
  }

  /* ── Vertical Stepper ── */
  .stepper {
    display: flex;
    flex-direction: column;
  }

  .stepper-item {
    display: flex;
    gap: 12px;
  }

  .stepper-rail {
    display: flex;
    flex-direction: column;
    align-items: center;
    flex-shrink: 0;
    width: 24px;
  }

  .stepper-icon {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }
  .stepper-icon-success { background: #15803D; }
  .stepper-icon-pending { background: #E5E7EB; }

  .check-pop-in {
    animation: checkIconPop 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
  }
  @keyframes checkIconPop {
    0%   { transform: scale(0); opacity: 0; }
    50%  { transform: scale(1.12); opacity: 1; }
    70%  { transform: scale(0.95); }
    100% { transform: scale(1); }
  }

  .line-grow {
    animation: lineGrow 0.4s cubic-bezier(0.4, 0, 0.2, 1) forwards;
    transform-origin: top center;
  }
  @keyframes lineGrow {
    0%   { transform: scaleY(0); opacity: 0; }
    100% { transform: scaleY(1); opacity: 1; }
  }

  .stepper-dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: #9CA3AF;
  }

  .stepper-line {
    width: 2px;
    flex: 1;
    min-height: 16px;
    margin: 4px 0;
  }
  .stepper-line-success { background: #15803D; }
  .stepper-line-pending { background: #E5E7EB; }

  .stepper-content {
    display: flex;
    flex-direction: column;
    gap: 4px;
    flex: 1;
    padding-bottom: 20px;
  }

  .stepper-item:last-child .stepper-content {
    padding-bottom: 0;
  }

  .step-title {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    line-height: 20px;
    color: #111827;
  }

  .step-desc {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 18px;
    color: #6B7280;
  }

  .step-desc strong {
    font-weight: 600;
    color: #111827;
  }

  /* Processing badge */
  .processing-badge {
    position: absolute;
    top: -2px;
    left: 50%;
    transform: translateX(-50%);
    background: linear-gradient(98deg, #0D0D0D 0%, #414141 100%);
    border: 1px solid #D1D5DB;
    border-radius: 0 0 16px 16px;
    padding: 6px 12px;
    white-space: nowrap;
  }

  .processing-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 10px;
    line-height: 16px;
    color: #FFFFFF;
    text-align: center;
  }

  /* ── Info box ── */
  .info-box {
    display: flex;
    gap: 8px;
    padding: 16px 12px;
    margin: 24px 16px 0;
    background: #F3F4F6;
    border: 1px solid #D1D5DB;
    border-radius: 8px;
    width: calc(100% - 32px);
    position: relative;
    z-index: 1;
  }

  .info-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    line-height: 132%;
    color: #111827;
    flex: 1;
  }

  /* ── Confetti ── */
  .confetti-container {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 432px;
    height: 480px;
    z-index: 10;
    pointer-events: none;
  }
</style>
