<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import StatusBar from '$lib/components/StatusBar.svelte';

  let selected = $state<'etb' | 'ntb' | null>(null);
  let loading = $state(false);

  async function handleStart() {
    if (!selected) return;
    loading = true;
    await new Promise(r => setTimeout(r, 500));
    loading = false;
    goto(`${base}/mobile`);
  }
</script>

<svelte:head><title>Are you a IOB customer? – IOB</title></svelte:head>

<div class="screen">

  <div class="main-content-container">

    <!-- ── HEADER ── -->
    <div class="header-container">
      <StatusBar />
      <div class="icon-row">
        <div></div>
        <div class="icon-group">
          <button class="icon-btn" aria-label="Help">
            <i class="ph-duotone ph-headset" style="font-size:24px; color:#111827"></i>
          </button>
        </div>
      </div>
      <div class="progress-bar-container">
        <div class="progress-bar-wrapper">
          <h1 class="screen-title">Are you already a customer of IOB Bank?</h1>
        </div>
      </div>
    </div>

    <!-- ── CARDS ── -->
    <div class="cards-section">
      <div class="cards-inner">

        <!-- CARD 1: ETB -->
        <button
          class="option-card"
          class:selected={selected === 'etb'}
          onclick={() => selected = 'etb'}
          aria-pressed={selected === 'etb'}
        >
          <div class="card-row">
            <div class="radio" class:checked={selected === 'etb'}>
              {#if selected === 'etb'}
                <div class="check-anim">
                  <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                    <circle cx="10" cy="10" r="10" fill="#15803D"/>
                    <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
                  </svg>
                </div>
              {/if}
            </div>
            <div class="card-text">
              <span class="card-title">I am a customer of IOB Bank</span>
              <p class="card-desc">
                You can be our customer through any other finanical partner or directly with IOB Bank
              </p>
            </div>
          </div>

          <!-- Toast — vertical gradient bg + horizontal green overlay -->
          <div class="toast toast-etb">
            <div class="toast-overlay toast-overlay-green"></div>
            <div class="toast-row">
              <img src="/bolt-green.svg" width="18" height="18" alt="" aria-hidden="true" style="flex-shrink:0" />
              <div class="toast-text-col">
                <p class="toast-msg toast-msg-green">
                  Your card application will be super easy
                  since we have your details with us
                </p>
              </div>
            </div>
          </div>
        </button>

        <!-- CARD 2: NTB -->
        <button
          class="option-card"
          class:selected={selected === 'ntb'}
          onclick={() => selected = 'ntb'}
          aria-pressed={selected === 'ntb'}
        >
          <div class="card-row card-row-center">
            <div class="radio" class:checked={selected === 'ntb'}>
              {#if selected === 'ntb'}
                <div class="check-anim">
                  <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                    <circle cx="10" cy="10" r="10" fill="#15803D"/>
                    <path d="M6.5 10.2L9 12.8L13.5 7.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
                  </svg>
                </div>
              {/if}
            </div>
            <div class="card-text">
              <span class="card-title">I am a new customer</span>
            </div>
          </div>

          <!-- Toast — vertical gradient bg + horizontal blue overlay -->
          <div class="toast toast-ntb">
            <div class="toast-overlay toast-overlay-blue"></div>
            <div class="toast-col">
              <div class="speed-chip">
                <img src="/bolt-purple.svg" width="18" height="18" alt="" aria-hidden="true" style="flex-shrink:0" />
                <span class="chip-label">Takes less than 5 minutes</span>
              </div>
              <div class="toast-text-col">
                <p class="toast-msg toast-msg-blue">
                  Glad you chose IOB Bank for your credit card
                </p>
              </div>
            </div>
          </div>
        </button>

      </div>
    </div>

  </div>

  <div class="spacer"></div>

  <!-- ── FOOTER ── -->
  <div class="footer">
    <div class="footer-inner">
      <button
        class="btn-primary"
        disabled={!selected || loading}
        onclick={handleStart}
      >
        {#if loading}
          <span class="spinner"></span>
        {:else}
          Start application
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

  .main-content-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    align-self: stretch;
    gap: 32px;
    flex-shrink: 0;
  }

  /* ── Header ── */
  .header-container {
    display: flex;
    flex-direction: column;
    gap: 24px;
    padding: 0 0 16px;
    width: 100%;
    background: #FFFCF4;
    flex-shrink: 0;
  }

  .icon-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
  }
  .icon-group { display: flex; align-items: center; }

  .icon-btn {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 48px;
    height: 48px;
    background: none;
    border: none;
    cursor: pointer;
    border-radius: 50%;
    transition: background 0.15s;
    flex-shrink: 0;
  }
  .icon-btn:active { background: rgba(0,0,0,0.06); }

  .progress-bar-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    align-self: stretch;
    gap: 4px;
  }
  .progress-bar-wrapper {
    display: flex;
    align-items: center;
    align-self: stretch;
    gap: 16px;
    padding: 0 16px;
  }
  .screen-title {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 24px;
    line-height: 1.2;
    color: #212121;
    letter-spacing: -0.25px;
    flex: 1;
  }

  /* ── Cards ── */
  .cards-section {
    display: flex;
    flex-direction: column;
    align-items: stretch;
    align-self: stretch;
    gap: 3px;
    padding: 0 16px;
    flex-shrink: 0;
  }
  .cards-inner {
    display: flex;
    flex-direction: column;
    align-items: stretch;
    align-self: stretch;
    gap: 24px;
  }

  .option-card {
    display: flex;
    flex-direction: column;
    align-self: stretch;
    background: #FFFCF4;
    border: 1px solid #D1D5DB;
    border-radius: 12px;
    cursor: pointer;
    text-align: left;
    padding: 0;
    overflow: hidden;
    transition: border-color 0.18s ease, box-shadow 0.18s ease;
  }
  .option-card.selected {
    border-color: #242A80;
  }
  .option-card:active { opacity: 0.93; }

  .card-row {
    display: flex;
    align-items: flex-start;
    align-self: stretch;
    gap: 8px;
    padding: 12px 12px 16px;
  }
  .card-row-center { align-items: center; }

  /* Radio — 20×20 Material Design */
  .radio {
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
  .radio.checked { border: none; background: none; }

  /* Subtle pop animation on check */
  .check-anim {
    animation: checkPop 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
    display: flex;
  }
  @keyframes checkPop {
    0%   { transform: scale(0); opacity: 0; }
    60%  { transform: scale(1.15); opacity: 1; }
    100% { transform: scale(1); }
  }
  .radio-dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: #184595;
  }

  .card-text {
    display: flex;
    flex-direction: column;
    gap: 3px;
    flex: 1;
  }
  .card-title {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    line-height: 1.32;
    color: #111827;
  }
  .card-desc {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 1.5;
    color: #6B7280;
  }

  /* ══════════════════════════════════════
     TOAST — updated per Figma 17:4012
     Vertical gradient bg: 0deg, #F3F4F6 34% → rgba(215,215,215,0) 100%
     Padding: 12px 8px, gap:8px, w:328px
  ══════════════════════════════════════ */
  .toast {
    position: relative;
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 12px 8px;
    width: 100%;
    overflow: hidden;
    align-self: stretch;
    /* Vertical gradient background — Figma fill_9G3K93 */
    background: linear-gradient(0deg, rgba(243,244,246,1) 34%, rgba(215,215,215,0) 100%);
  }

  /* Horizontal overlay gradient (absolute, behind content) */
  .toast-overlay {
    position: absolute;
    inset: 0;
    pointer-events: none;
  }

  /* ETB: green horizontal overlay — Figma fill_OME5R5 */
  .toast-overlay-green {
    background: linear-gradient(90deg,
      rgba(240,253,244,1) 0%,
      rgba(247,251,246,0.54) 46%,
      rgba(255,248,248,0) 100%);
  }

  /* NTB: blue horizontal overlay — Figma fill_SUAS9Y */
  .toast-overlay-blue {
    background: linear-gradient(90deg,
      rgba(236,242,255,1) 41%,
      rgba(247,251,246,0.54) 71%,
      rgba(255,248,248,0) 100%);
  }

  /* ETB toast content — row, gap:4px, px:12px */
  .toast-row {
    position: relative;
    display: flex;
    align-items: center;
    gap: 4px;
    width: 100%;
    padding: 0 12px;
  }

  /* NTB toast content — column, gap:8px, px:12px */
  .toast-col {
    position: relative;
    display: flex;
    flex-direction: column;
    gap: 8px;
    width: 100%;
    padding: 0 12px;
  }

  .toast-text-col {
    display: flex;
    flex-direction: column;
    gap: 4px;
    flex: 1;
  }

  .toast-msg {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 12px;
    line-height: 1.5;
    width: 100%;
  }
  .toast-msg-green { color: #15803D; }
  /* NTB message — dark navy blue — Figma fill_CGP3S8 */
  .toast-msg-blue  { color: #242A80; }

  /* Speed chip — bg:#DDE8FF, text:#242A80, semibold 12px */
  .speed-chip {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 4px;
    padding: 4px 8px;
    background: #DDE8FF;
    border-radius: 500px;
    width: fit-content;
  }
  .chip-label {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 12px;
    line-height: 1.5;
    color: #242A80;
  }

  /* ── Spacer ── */
  .spacer { flex: 1; }

  /* ── Footer ── */
  .footer { position: sticky; bottom: 0; z-index: 5; margin-top: auto;
    padding: 16px 0;
    background: #FFFCF4;
    flex-shrink: 0;
    width: 100%;
  }
  .footer-inner { padding: 0 16px; }

  /* Button — dark navy blue #242A80 — Figma fill_CGP3S8 */
  .btn-primary {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 48px;
    background: #184595;
    color: #FFFFFF;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 16px;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    box-shadow: 0px 4px 0px #06142A;
    transition: opacity 0.15s, transform 0.1s;
  }
  .btn-primary:active:not(:disabled) {
    opacity: 0.88;
    transform: scale(0.99);
  }
  .btn-primary:disabled {
    background: #D1D5DB;
    box-shadow: none;
    cursor: not-allowed;
  }

  .spinner {
    width: 20px;
    height: 20px;
    border: 2.5px solid rgba(255,255,255,0.4);
    border-top-color: #FFFFFF;
    border-radius: 50%;
    animation: spin 0.7s linear infinite;
  }
  @keyframes spin { to { transform: rotate(360deg); } }
</style>
