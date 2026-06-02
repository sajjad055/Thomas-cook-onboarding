<!--
  ToastMessage.svelte — Toast notification component
  States: success, pending (error in Figma = pending here), error (failed in Figma)
  
  Props:
    - state: 'success' | 'pending' | 'error'
    - message: string
    - visible: boolean
    - onclose: () => void (optional)
-->
<script lang="ts">
  import { fade, fly } from 'svelte/transition';

  let {
    state = 'success',
    message = '',
    visible = false,
    onclose = undefined,
  }: {
    state?: 'success' | 'pending' | 'error';
    message?: string;
    visible?: boolean;
    onclose?: (() => void) | undefined;
  } = $props();
</script>

{#if visible}
  <div
    class="toast toast-{state}"
    in:fly={{ y: -20, duration: 300 }}
    out:fade={{ duration: 200 }}
    role="alert"
  >
    <!-- Gradient background -->
    <div class="toast-gradient"></div>

    <!-- Content -->
    <div class="toast-content">
      <div class="toast-icon-text">
        <!-- State icon -->
        <div class="feedback-icon">
          {#if state === 'success'}
            <div class="icon-circle icon-circle-success">
              <i class="ph ph-check" style="font-size:12px; color:#15803D"></i>
            </div>
          {:else if state === 'error'}
            <div class="icon-circle icon-circle-error">
              <i class="ph ph-x" style="font-size:12px; color:#B91C1C"></i>
            </div>
          {:else}
            <div class="icon-circle icon-circle-pending">
              <i class="ph ph-info" style="font-size:14px; color:#B45309"></i>
            </div>
          {/if}
        </div>

        <!-- Message -->
        <div class="toast-message-wrap">
          <p class="toast-text">{message}</p>
        </div>
      </div>

      <!-- Close button -->
      {#if onclose}
        <button class="toast-close" onclick={onclose} aria-label="Dismiss">
          <i class="ph ph-x" style="font-size:15px; color:#111827"></i>
        </button>
      {/if}
    </div>
  </div>
{/if}

<style>
  .toast {
    position: relative;
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 8px;
    width: 100%;
    background: #F3F4F6;
    border: 1px solid #D1D5DB;
    border-radius: 8px;
    overflow: hidden;
  }

  /* Gradient backgrounds per state */
  .toast-gradient {
    position: absolute;
    inset: 0;
    pointer-events: none;
  }

  .toast-success .toast-gradient {
    background: linear-gradient(93deg, rgba(240, 253, 244, 1) 0%, rgba(255, 255, 255, 0) 100%);
  }

  .toast-error .toast-gradient {
    background: linear-gradient(93deg, rgba(254, 242, 242, 1) 0%, rgba(255, 255, 255, 0) 100%);
  }

  .toast-pending .toast-gradient {
    background: linear-gradient(90deg, rgba(255, 251, 235, 1) 0%, rgba(255, 248, 248, 0) 100%);
  }

  /* Content layer */
  .toast-content {
    position: relative;
    display: flex;
    align-items: center;
    gap: 4px;
    width: 100%;
    z-index: 1;
  }

  .toast-icon-text {
    display: flex;
    align-items: center;
    gap: 4px;
    flex: 1;
  }

  /* Feedback icon */
  .feedback-icon {
    flex-shrink: 0;
    width: 20px;
    height: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .icon-circle {
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
    padding: 2.5px;
  }

  .icon-circle-success {
    background: none;
  }

  .icon-circle-error {
    background: none;
  }

  .icon-circle-pending {
    background: #FFFFFF;
    border-radius: 500px;
  }

  /* Message text */
  .toast-message-wrap {
    flex: 1;
  }

  .toast-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 12px;
    line-height: 18px;
    color: #111827;
  }

  .toast-success .toast-text {
    color: #15803D;
  }

  .toast-error .toast-text {
    color: #B91C1C;
  }

  .toast-pending .toast-text {
    color: #B45309;
  }

  /* Close button */
  .toast-close {
    flex-shrink: 0;
    width: 20px;
    height: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: none;
    border: none;
    cursor: pointer;
    border-radius: 50%;
    padding: 2.4px;
  }
</style>
