<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import { onMount, onDestroy } from 'svelte';
  import { fade, scale } from 'svelte/transition';
  import { cubicOut } from 'svelte/easing';
  import { browser } from '$app/environment';
  import StatusBar from '$lib/components/StatusBar.svelte';

  // ── Status text rotation ────────────────────────────────────────────
  const statusMessages = [
    'Checking your income details',
    'Analysing your bank transactions',
    'Identifying salary credits',
    'Detecting recurring patterns',
    'Calculating your spending behaviour',
    'Preparing your offer',
    'Almost there',
  ];
  let statusIndex = $state(0);

  // ── Lottie ──────────────────────────────────────────────────────────
  let lottieReady = $state(false);

  // ── Game state ─────────────────────────────────────────────────────
  // The track is wider than the frame. The frame stays centered. The card
  // (same size as frame) slides L↔R across the track, passing through the
  // frame. User taps when the card is centered in the frame.
  let trackEl: HTMLDivElement;
  let frameEl: HTMLDivElement;
  let cardEl: HTMLDivElement;
  let trackWidth = $state(0);
  let frameWidth = $state(0);
  let cardWidth = $state(0);

  // Card travels from -cardWidth to trackWidth (off-screen on each side)
  let travelDistance = $derived(trackWidth + cardWidth);
  // 0 = card just off-screen left ; 1 = card just off-screen right
  let cardX = $state(0);

  type FlashState = 'idle' | 'hit' | 'miss';
  let flashState = $state<FlashState>('idle');
  let pointsEarned = $state(false);
  let showRewardPopup = $state(false);
  let isPressed = $state(false); // press-and-hold state
  let isPaused = $state(false);  // game paused after first successful catch

  // Hit when card center is within ±halfWindow of track/frame center.
  // ~22% of frame width gives ~45% win rate at this speed.
  const HIT_WINDOW_RATIO = 0.22;
  const SWEEP_DURATION = 650; // fast horizontal travel

  // ── Animation loop ──────────────────────────────────────────────────
  let rafId: number | null = null;
  let direction = 1;
  let lastFlipTime = 0;
  // Tracks how much "elapsed" was already accumulated before a pause.
  // While pressed, lastFlipTime gets shifted forward each frame so the
  // card stays at exactly its current position when released.
  function animate(now: number) {
    if (!lastFlipTime) lastFlipTime = now;

    if (isPressed || isPaused) {
      // Freeze: keep advancing lastFlipTime so resume is seamless.
      lastFlipTime = now - (cardX === 0 ? 0 : (() => {
        const startX = direction === 1 ? -cardWidth : trackWidth;
        const endX = direction === 1 ? trackWidth : -cardWidth;
        const totalDistance = endX - startX;
        const traveled = (cardX - startX) / totalDistance;
        return Math.max(0, Math.min(1, traveled)) * SWEEP_DURATION;
      })());
      rafId = requestAnimationFrame(animate);
      return;
    }

    const elapsed = now - lastFlipTime;
    let t = Math.min(1, elapsed / SWEEP_DURATION);
    const eased = t; // linear

    const startX = direction === 1 ? -cardWidth : trackWidth;
    const endX = direction === 1 ? trackWidth : -cardWidth;
    cardX = startX + (endX - startX) * eased;

    if (t >= 1) {
      direction *= -1;
      lastFlipTime = now;
    }
    rafId = requestAnimationFrame(animate);
  }

  // ── Press / release handlers ────────────────────────────────────────
  function evaluateAttempt() {
    // Card is centered in frame when card center = track center
    const cardCenter = cardX + cardWidth / 2;
    const trackCenter = trackWidth / 2;
    const offset = Math.abs(cardCenter - trackCenter);
    const halfHitWindow = (frameWidth * HIT_WINDOW_RATIO) / 2;
    const isHit = offset <= halfHitWindow;

    if (isHit) {
      flashState = 'hit';
      // Snap the card precisely to centre so it sits perfectly inside frame
      cardX = trackCenter - cardWidth / 2;

      if (!pointsEarned) {
        // First successful catch — pause the game and show reward
        pointsEarned = true;
        isPaused = true;
        setTimeout(() => { showRewardPopup = true; }, 220);
      }
    } else {
      flashState = 'miss';
    }
  }

  function handlePressStart(e: Event) {
    e.preventDefault();
    if (isPressed || isPaused) return;
    isPressed = true;
    evaluateAttempt();
  }

  function handlePressEnd() {
    if (!isPressed) return;
    isPressed = false;
    // Keep flash state on if we just earned points (paused state takes over)
    if (!isPaused) flashState = 'idle';
  }

  // Resume the game from the paused state (after the first catch)
  function continuePlaying() {
    isPaused = false;
    isPressed = false;
    flashState = 'idle';
  }

  function dismissPopup() { showRewardPopup = false; }

  // ── Lifecycle ───────────────────────────────────────────────────────
  let statusTimer: ReturnType<typeof setInterval>;
  let navTimer: ReturnType<typeof setTimeout>;

  onMount(async () => {
    if (browser) {
      try { await import('@lottiefiles/dotlottie-wc'); } catch {}
    }
    lottieReady = true;

    requestAnimationFrame(() => {
      if (trackEl) trackWidth = trackEl.clientWidth;
      if (frameEl) frameWidth = frameEl.clientWidth;
      if (cardEl) cardWidth = cardEl.clientWidth;
      rafId = requestAnimationFrame(animate);
    });

    statusTimer = setInterval(() => {
      if (statusIndex < statusMessages.length - 1) statusIndex++;
    }, 4200);

    navTimer = setTimeout(() => {
      goto(`${base}/verification`);
    }, 30000);
  });

  onDestroy(() => {
    if (rafId !== null) cancelAnimationFrame(rafId);
    clearInterval(statusTimer);
    clearTimeout(navTimer);
  });
</script>

<svelte:head><title>Checking your income — IOB</title></svelte:head>

<div class="screen">
  <StatusBar />

  <!-- ── TOP: Lottie + status text ─────────────────────────────── -->
  <div class="loader-block">
    {#if lottieReady}
      <div class="lottie-wrap">
        <dotlottie-wc
          src="{base}/Document.lottie"
          autoplay
          loop
          speed="1"
          style="width: 64px; height: 64px;"
        ></dotlottie-wc>
      </div>
    {:else}
      <div class="lottie-wrap"></div>
    {/if}

    <div class="status-text-wrap">
      {#key statusIndex}
        <p class="status-main" in:fade={{ duration: 400, delay: 80 }}>
          {statusMessages[statusIndex]}
        </p>
      {/key}
    </div>
  </div>

  <!-- ── MIDDLE: Game with fixed center frame, card flies across ─── -->
  <div class="game-block">
    <button
      class="game-track"
      bind:this={trackEl}
      onpointerdown={handlePressStart}
      onpointerup={handlePressEnd}
      onpointerleave={handlePressEnd}
      onpointercancel={handlePressEnd}
      oncontextmenu={(e) => e.preventDefault()}
      aria-label="Press and hold to catch the card"
    >
      <!-- Sliding card travels behind the frame across the full track -->
      <div class="game-card" bind:this={cardEl} style="transform: translateX({cardX}px);">
        <img src="{base}/tc.svg" alt="" draggable="false" />
      </div>

      <!-- Green glow halo behind the frame on success -->
      {#if isPaused}
        <div class="frame-halo" aria-hidden="true" in:fade={{ duration: 280 }}></div>
      {/if}

      <!-- Fixed centered frame on top -->
      <div
        class="game-frame"
        class:flash-miss={flashState === 'miss'}
        bind:this={frameEl}
        aria-hidden="true"
      >
        <img
          src="{base}/{isPaused ? 'tc-frame-success.svg' : flashState === 'hit' ? 'tc-frame-hit.svg' : flashState === 'miss' ? 'tc-frame-miss.svg' : 'tc-frame.svg'}"
          alt=""
          draggable="false"
        />
      </div>
    </button>

    <!-- Pill CTA — turns into "Continue playing" button after first catch -->
    {#if isPaused}
      <button class="game-pill game-pill-action" onclick={continuePlaying}>
        Continue playing
      </button>
    {:else}
      <div class="game-pill">Press &amp; hold to catch the card</div>
    {/if}

    <!-- Caption + coin chip -->
    <div class="game-caption-wrap">
      <p class="game-caption">While you wait, catch the card in the frame and earn</p>
      <div class="coin-chip">
        <img src="{base}/coin.svg" alt="" class="coin-chip-icon" draggable="false" />
        {#if pointsEarned}
          <span class="coin-chip-text"><span class="coin-prefix">You earned</span> <strong>10</strong> points</span>
        {:else}
          <span class="coin-chip-text"><strong>10</strong> points</span>
        {/if}
      </div>
    </div>
  </div>

  <!-- ── Reward popup ──────────────────────────────────────────── -->
  {#if showRewardPopup}
    <button
      class="popup-backdrop"
      onclick={dismissPopup}
      aria-label="Dismiss reward popup"
      in:fade={{ duration: 200 }}
      out:fade={{ duration: 200 }}
    ></button>
    <div
      class="reward-popup"
      role="dialog"
      aria-live="polite"
      in:scale={{ duration: 320, easing: cubicOut, start: 0.85 }}
      out:fade={{ duration: 200 }}
    >
      <div class="reward-icon">
        <img src="{base}/coin.svg" alt="" class="reward-coin" />
      </div>
      <h3 class="reward-title">You earned 10 points</h3>
      <p class="reward-sub">Reward will be credited to your IOB rewards balance.</p>
      <button class="reward-btn" onclick={dismissPopup}>Keep playing</button>
    </div>
  {/if}
</div>

<style>
  .screen {
    width: 100%;
    min-height: 100dvh;
    display: flex;
    flex-direction: column;
    background: #FFFDF6;
    padding: 0 16px 24px;
    position: relative;
    overflow: hidden;
  }

  /* ── Loader block (top) ───────────────────────────────────────── */
  .loader-block {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 16px;
    padding: 60px 0 40px;
    flex-shrink: 0;
  }

  .lottie-wrap {
    width: 64px;
    height: 64px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .status-text-wrap {
    min-height: 22px;
    text-align: center;
  }

  .status-main {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 300;
    font-size: 14px;
    line-height: 22px;
    color: #111827;
    text-shadow: 0.5px 0.75px 0px rgba(0, 0, 0, 0.05);
  }

  /* ── Game block (middle/bottom) ───────────────────────────────── */
  .game-block {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 24px;
    flex: 1;
    justify-content: center;
    padding-bottom: 32px;
  }

  /* Game track — full screen-width strip the card travels across */
  .game-track {
    position: relative;
    width: 100%;
    aspect-ratio: 190 / 245;
    max-height: 360px;
    background: transparent;
    border: none;
    padding: 0;
    cursor: pointer;
    overflow: hidden;
    -webkit-tap-highlight-color: transparent;
    user-select: none;
    -webkit-user-select: none;
    touch-action: none;
  }

  /* Fixed centered frame — uses tc-frame.svg (same silhouette as card) */
  .game-frame {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 60%;
    aspect-ratio: 190 / 245;
    transform: translate(-50%, -50%);
    pointer-events: none;
    z-index: 2;
  }

  .game-frame img {
    width: 100%;
    height: 100%;
    display: block;
  }

  .game-frame.flash-miss {
    animation: shake 0.32s cubic-bezier(.36,.07,.19,.97);
  }

  @keyframes shake {
    10%, 90% { transform: translate(calc(-50% - 2px), -50%); }
    20%, 80% { transform: translate(calc(-50% + 3px), -50%); }
    30%, 50%, 70% { transform: translate(calc(-50% - 5px), -50%); }
    40%, 60% { transform: translate(calc(-50% + 5px), -50%); }
  }

  /* Sliding card — same size as frame, travels across the track */
  .game-card {
    position: absolute;
    top: 50%;
    left: 0;
    width: 60%;
    aspect-ratio: 190 / 245;
    translate: 0 -50%;
    pointer-events: none;
    will-change: transform;
    z-index: 1;
  }

  .game-card img {
    width: 100%;
    height: 100%;
    display: block;
    border-radius: 8px;
    overflow: hidden;
  }

  /* Pill CTA hint below the frame */
  .game-pill {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    background: #F9FAFB;
    border: 1px solid #D1D5DB;
    border-radius: 32px;
    padding: 3px 12px;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 300;
    font-size: 12px;
    line-height: 22px;
    color: #111827;
    text-shadow: 0.5px 0.75px 0px rgba(0, 0, 0, 0.05);
  }

  /* Caption + coin chip */
  .game-caption-wrap {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 16px;
    max-width: 280px;
    text-align: center;
  }

  .game-caption {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 500;
    font-size: 18px;
    line-height: 22px;
    letter-spacing: 0.5px;
    color: #111827;
    text-shadow: 0.5px 0.75px 0px rgba(0, 0, 0, 0.05);
  }

  .coin-chip {
    display: inline-flex;
    align-items: center;
    gap: 4px;
    background: linear-gradient(47deg, #FFFBEB 0%, #FFF7D8 100%);
    backdrop-filter: blur(4px);
    -webkit-backdrop-filter: blur(4px);
    border-radius: 999px;
    padding: 6px 12px;
  }

  .coin-chip-icon {
    width: 24px;
    height: 24px;
    flex-shrink: 0;
  }

  .coin-chip-text {
    font-family: 'Nunito Sans', sans-serif;
    font-size: 16px;
    line-height: 22px;
    color: #D89F12;
  }
  .coin-chip-text strong {
    font-weight: 700;
    font-size: 20px;
  }

  /* ── Reward popup ─────────────────────────────────────────────── */
  .popup-backdrop {
    position: fixed;
    inset: 0;
    background: rgba(17, 24, 39, 0.55);
    z-index: 200;
    border: none;
    padding: 0;
    cursor: pointer;
  }

  .reward-popup {
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    width: calc(100% - 48px);
    max-width: 320px;
    background: #FFFFFF;
    border-radius: 20px;
    padding: 28px 24px 20px;
    z-index: 201;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 12px;
    box-shadow: 0 24px 64px rgba(6, 20, 42, 0.32);
    text-align: center;
  }

  .reward-icon { display: flex; }
  .reward-coin { width: 56px; height: 56px; }

  .reward-title {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 700;
    font-size: 20px;
    line-height: 1.3;
    color: #111827;
  }

  .reward-sub {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 13px;
    line-height: 1.5;
    color: #6B7280;
  }

  .reward-btn {
    margin-top: 8px;
    width: 100%;
    height: 44px;
    background: #184595;
    color: #FFFFFF;
    border: none;
    border-radius: 8px;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 15px;
    cursor: pointer;
    box-shadow: 0px 3px 0px #06142A;
    transition: opacity 0.15s, transform 0.1s;
  }
  .reward-btn:active { opacity: 0.9; transform: scale(0.98); }
</style>
