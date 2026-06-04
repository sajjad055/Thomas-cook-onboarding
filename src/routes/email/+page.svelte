<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import { fade } from 'svelte/transition';
  import { onMount } from 'svelte';
  import StatusBar from '$lib/components/StatusBar.svelte';

  let email = $state('');
  let touched = $state(false);
  let loading = $state(false);
  let focused = $state(false);

  onMount(() => {
    setTimeout(() => {
      const el = document.getElementById('email-input');
      el?.focus();
      focused = true;
    }, 400);
  });

  let isValid = $derived(/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email));
  let showError = $derived(touched && email.length > 0 && !isValid);

  function onBlur() {
    touched = true;
    focused = false;
  }

  async function handleGetOTP() {
    touched = true;
    if (!isValid) return;
    loading = true;
    await new Promise(r => setTimeout(r, 600));
    loading = false;
    goto(`${base}/email/otp`);
  }
</script>

<svelte:head><title>Verify email – IOB</title></svelte:head>

<div class="screen">

  <div class="header-area">
    <StatusBar />
    <div class="icon-row">
      <button class="icon-btn" aria-label="Back" onclick={() => goto(`${base}/personal-details`)}>
        <i class="ph ph-caret-left" style="font-size:24px; color:#111827"></i>
      </button>
      <div class="icon-group">
        <button class="icon-btn" aria-label="Help">
          <i class="ph-duotone ph-headset" style="font-size:24px; color:#111827"></i>
        </button>
      </div>
    </div>

    <div class="illustration-frame">
      <img src="{base}/security.svg" alt="" class="illustration-img" />
    </div>

    <div class="progress-row">
      <h1 class="screen-title">Verify your email</h1>
    </div>
  </div>

  <div class="form-area">
    <div class="form-inner">
      <div class="field-group">
        <!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
        <div
          class="tf"
          class:active={focused || email.length > 0}
          class:focused
          class:error={showError}
          onclick={() => { const el = document.getElementById('email-input'); el?.focus(); }}
        >
          <div class="tf-content" class:active={focused || email.length > 0}>
            <span class="tf-label" class:floated={focused || email.length > 0}>Enter email address</span>
            <div class="email-row" class:shown={focused || email.length > 0}>
              <input
                id="email-input"
                type="email"
                inputmode="email"
                autocomplete="email"
                placeholder={focused ? 'name@example.com' : ''}
                value={email}
                oninput={(e) => { email = (e.target as HTMLInputElement).value; }}
                onblur={onBlur}
                onfocus={() => focused = true}
                class="email-input"
              />
            </div>
          </div>
        </div>

        {#if showError}
          <p class="tf-helper tf-error" in:fade={{ duration: 150 }}>
            Please enter a valid email address
          </p>
        {/if}
      </div>
    </div>
  </div>

  <div class="spacer"></div>

  <div class="footer">
    <button
      class="btn-primary"
      class:loading
      disabled={loading}
      onclick={handleGetOTP}
    >
      {#if loading}
        <span class="spinner"></span>
      {:else}
        Get OTP
      {/if}
    </button>
  </div>

</div>

<style>
  .screen {
    width: 100%; min-height: 100dvh; display: flex;
    flex-direction: column; background: #FFFCF4;
  }

  .header-area {
    background: #FFFCF4; padding-bottom: 0; flex-shrink: 0;
    display: flex; flex-direction: column; gap: 24px;
  }

  .icon-row { display: flex; align-items: center; justify-content: space-between; padding: 0; width: 100%; }
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
    line-height: 1.2; color: #212121; letter-spacing: -0.25px; flex: 1;
  }

  .form-area {
    background: #FFFCF4; border-radius: 16px 16px 0 0;
    padding: 24px 0 0; flex-shrink: 0;
  }
  .form-inner { display: flex; flex-direction: column; gap: 16px; padding: 0 16px; }
  .field-group { display: flex; flex-direction: column; gap: 6px; }

  .tf {
    display: flex; align-items: center; gap: 16px;
    background: #FFFFFF; border: 1px solid #D1D5DB; border-radius: 8px;
    padding: 10px 12px; min-height: 52px; cursor: text;
    transition: border-color 0.15s ease;
  }
  .tf.focused { border-color: #D1D5DB; }
  .tf.error { border-color: #B91C1C; }

  .tf-content { display: flex; flex-direction: column; flex: 1; min-width: 0; justify-content: center; }
  .tf-content.active { gap: 4px; }

  .tf-label {
    font-family: 'Nunito Sans', sans-serif; font-weight: 400; font-size: 12px;
    line-height: 1.5; color: #6B7280; pointer-events: none; transition: color 0.18s ease;
  }
  .tf-label.floated { color: #111827; }

  .email-row {
    display: flex; align-items: center; gap: 8px;
    height: 0; opacity: 0; overflow: hidden;
    transition: height 0.15s ease, opacity 0.15s ease;
  }
  .email-row.shown { height: 20px; opacity: 1; }

  .email-input {
    flex: 1; background: none; border: none; outline: none;
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 14px;
    color: #111827; padding: 0; line-height: 1.43;
  }
  .email-input::placeholder { color: #9CA3AF; font-weight: 400; }

  .tf-helper {
    font-family: 'Nunito Sans', sans-serif; font-weight: 400; font-size: 12px;
    line-height: 1.5; color: #6B7280; padding: 0 4px;
  }
  .tf-error { color: #B91C1C; }

  .spacer { flex: 1; }

  .footer {
    position: sticky; bottom: 0; z-index: 5; margin-top: auto;
    padding: 16px 16px calc(16px + env(safe-area-inset-bottom));
    background: #FFFCF4; flex-shrink: 0;
  }

  .btn-primary {
    display: flex; align-items: center; justify-content: center;
    width: 100%; height: 48px; background: #184595; color: #FFFFFF;
    font-family: 'Nunito Sans', sans-serif; font-weight: 600; font-size: 16px;
    border: none; border-radius: 8px; cursor: pointer;
    box-shadow: 0px 4px 0px #06142A; transition: opacity 0.15s ease, transform 0.1s ease;
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
