<script lang="ts">
  import { onMount } from 'svelte';
  import { fly, fade } from 'svelte/transition';
  import { cubicOut } from 'svelte/easing';
  import { browser } from '$app/environment';
  import { base } from '$app/paths';
  import StatusBar from '$lib/components/StatusBar.svelte';
  import ShimmerText from '$lib/effects/ShimmerText.svelte';

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
  let lineProgress = $state(0); // 0 to 100, animates the line between step 1 and 2

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

      // Cascade content smoothly
      setTimeout(() => showHeading = true, 600);
      setTimeout(() => {
        showSteps = true;
        showStep1 = true;
        showStep2 = true;
        showStep3 = true;
      }, 1000);
      // After stepper loads, tick step 1 and animate progress line
      setTimeout(() => showStep1Check = true, 2000);
      setTimeout(() => {
        showStep1Line = true;
        let progress = 0;
        const interval = setInterval(() => {
          progress += 3;
          lineProgress = progress;
          if (progress >= 50) clearInterval(interval);
        }, 30);
      }, 2400);
      setTimeout(() => showInfo = true, 3200);
    }, 1500);
  });
</script>

<svelte:head><title>Getting your card ready – IOB</title></svelte:head>

<div class="screen">

  <!-- Blurred gradient bg ellipse -->
  <div class="bg-ellipse" aria-hidden="true"></div>

  <!-- Rotating bg lighting — moves with the card -->
  <div class="bg-card-group" class:centered={phase === 'intro'} class:settled={phase === 'reveal'}>
    <img src="{base}/bg-lighting.svg" alt="" class="bg-lighting" draggable="false" aria-hidden="true" />
    <div class="card-float-wrap">
    <img src="{base}/tc-card-bank.svg" alt="TC Travel Card" class="card-img" draggable="false" />
      <span class="card-name-overlay">SAJJAD S</span>
    </div>
    <div class="card-shadow-element"></div>
  </div>

  <!-- Confetti burst during intro -->
  {#if showConfetti}
    <div class="confetti-container" aria-hidden="true">
      <dotlottie-wc
        src="{base}/confetti.lottie"
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

          <div class="h-stepper">

            <!-- Step 1: Done -->
            {#if showStep1}
              <div class="h-step h-step-done" in:fade={{ duration: 400 }}>
                <div class="h-step-top">
                  <div class="h-line h-line-invisible"></div>
                  {#if showStep1Check}
                    <div class="h-dot h-dot-done check-pop-in">
                      <i class="ph ph-check" style="font-size:10px; color:#FFFFFF"></i>
                    </div>
                  {:else}
                    <div class="h-dot h-dot-pending"></div>
                  {/if}
                  <div class="h-line h-line-animated" style="background: linear-gradient(90deg, #15803D {lineProgress}%, #E5E7EB {lineProgress}%)"></div>
                </div>
                <p class="h-step-label">Video KYC</p>
              </div>
            {/if}

            <!-- Step 2: Pending -->
            {#if showStep2}
              <div class="h-step">
                <div class="h-step-top">
                  <div class="h-line h-line-pending"></div>
                  <div class="h-dot h-dot-pending"></div>
                  <div class="h-line h-line-pending"></div>
                </div>
                <p class="h-step-label">Bank verification</p>
              </div>
            {/if}

            <!-- Step 3: Pending -->
            {#if showStep3}
              <div class="h-step">
                <div class="h-step-top">
                  <div class="h-line h-line-pending"></div>
                  <div class="h-dot h-dot-pending"></div>
                  <div class="h-line h-line-invisible"></div>
                </div>
                <p class="h-step-label">Card setup</p>
              </div>
            {/if}

          </div>

          <!-- Processing badge -->
          <ShimmerText trigger={true} loop={true} duration={2000}>
            <div class="processing-badge">
              <span class="processing-text">Card will be processed shortly</span>
            </div>
          </ShimmerText>

        </div>
      </div>
    {/if}

    <!-- Info box -->
    {#if showInfo}
      <div class="spacer-bottom"></div>
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
    background: #E8F0FF;
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
    background: linear-gradient(90deg, #E8F0FF 0%, #E8F0FF 100%);
    filter: blur(25.6px);
    pointer-events: none;
    z-index: 0;
    opacity: 0;
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
    flex-direction: column;
    align-items: center;
    justify-content: center;
    will-change: transform, opacity;
    transition: transform 1s cubic-bezier(0.25, 0.1, 0.25, 1), opacity 0.6s ease;
  }

  /* Phase 1: centered vertically */
  .bg-card-group.centered {
    transform: translateY(calc(50vh - 161px));
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
    filter: saturate(0.3) brightness(1.2) hue-rotate(200deg);
  }
  @keyframes bgRotate {
    from { transform: rotate(0deg); }
    to   { transform: rotate(360deg); }
  }

  .card-float-wrap {
    position: relative;
    width: 346px;
    height: 230px;
    animation: cardFloat 3.5s ease-in-out infinite;
    z-index: 1;
    margin: 0 auto;
  }

  .card-img {
    width: 346px;
    height: 230px;
    display: block;
    user-select: none;
    pointer-events: none;
  }

  .card-name-overlay {
    position: absolute;
    left: 101px;
    top: 142.5px;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 700;
    font-size: 7px;
    line-height: 1.32;
    letter-spacing: 0.15em;
    color: rgba(255,255,255,0.9);
    pointer-events: none;
    user-select: none;
    transform: rotate(15.7deg);
    transform-origin: left center;
    z-index: 2;
  }

  .card-shadow-element {
    width: 160px;
    height: 10px;
    border-radius: 50%;
    background: rgba(0, 48, 129, 0.25);
    filter: blur(3px);
    margin: 16px auto 0;
    animation: shadowScale 3.5s ease-in-out infinite;
    position: relative;
    z-index: 2;
  }

  @keyframes cardFloat {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-14px); }
  }

  @keyframes shadowScale {
    0%, 100% { transform: scaleX(1); opacity: 0.7; }
    50% { transform: scaleX(0.55); opacity: 0.2; }
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
    height: 320px;
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
    text-shadow: 1px 1px 0px rgba(0, 0, 0, 0.15);
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
    background: #FFFFFF;
    border: 0.5px solid #D1D5DB;
    border-radius: 8px;
    padding: 16px 12px;
    display: flex;
    flex-direction: column;
    gap: 24px;
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
  /* ── Horizontal line stepper ── */
  .h-stepper {
    display: flex;
    align-items: flex-start;
    overflow-x: auto;
    scrollbar-width: none;
    -webkit-overflow-scrolling: touch;
  }
  .h-stepper::-webkit-scrollbar { display: none; }

  .h-step {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 8px;
    min-width: 100px;
    flex: 1;
  }

  .h-step-top {
    display: flex;
    align-items: center;
    width: 100%;
  }

  .h-line {
    flex: 1;
    height: 2px;
    border-radius: 1px;
  }

  .h-line-done {
    background: #15803D;
  }

  .h-line-pending {
    background: #E5E7EB;
  }

  .h-line-invisible {
    background: transparent;
  }

  .h-line-animated {
    flex: 1;
    height: 2px;
    border-radius: 1px;
    transition: background 0.1s ease;
  }

  .h-dot {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }

  .h-dot-done {
    background: #15803D;
  }

  .h-dot-pending {
    background: #E5E7EB;
  }

  .h-step-label {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 12px;
    color: #111827;
    text-align: center;
    line-height: 1.4;
  }

  .h-step-done .h-step-label {
    color: #15803D;
  }

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
    position: relative;
    align-self: center;
    background: linear-gradient(98deg, #0D0D0D 0%, #414141 100%);
    border: 1px solid #D1D5DB;
    border-radius: 20px;
    padding: 6px 12px;
    white-space: nowrap;
  }

  .processing-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 12px;
    line-height: 18px;
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
