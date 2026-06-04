<!--
  BottomSheet.svelte
  ==================
  Reusable bottom sheet component matching Figma node 80:4197.

  Props:
    open        — controls visibility (bindable)
    title       — optional heading text inside the sheet
    onclose     — optional callback when sheet is dismissed

  Slots:
    default     — content area (list items, cards, forms, anything)
    footer      — sticky footer area (CTA buttons etc.)

  Usage:
    bind:open={showSheet} title="Accept terms & conditions"
    then render your content and a footer snippet inside.

  Animation:
    - Sheet slides up from bottom with cubicOut easing
    - Backdrop fades in simultaneously
    - Dismiss on backdrop click or programmatically via bind:open
-->
<script lang="ts">
  import { fly, fade } from 'svelte/transition';
  import { cubicOut } from 'svelte/easing';

  let {
    open = $bindable(false),
    title = '',
    onclose = undefined,
    children,
    footer,
  }: {
    open?: boolean;
    title?: string;
    onclose?: (() => void) | undefined;
    children?: any;
    footer?: any;
  } = $props();

  function dismiss() {
    open = false;
    onclose?.();
  }

  // Trap scroll on body when open
  $effect(() => {
    if (typeof document !== 'undefined') {
      document.body.style.overflow = open ? 'hidden' : '';
    }
    return () => {
      if (typeof document !== 'undefined') {
        document.body.style.overflow = '';
      }
    };
  });
</script>

{#if open}
  <!-- Backdrop -->
  <div
    class="backdrop"
    role="presentation"
    in:fade={{ duration: 250 }}
    out:fade={{ duration: 200 }}
    onclick={dismiss}
  ></div>

  <!-- Sheet -->
  <div
    class="sheet"
    role="dialog"
    aria-modal="true"
    aria-label={title || 'Bottom sheet'}
    in:fly={{ y: 500, duration: 380, easing: cubicOut }}
    out:fly={{ y: 500, duration: 280, easing: cubicOut }}
  >
    <!-- Handle -->
    <div class="handle-wrap" aria-hidden="true">
      <div class="handle"></div>
    </div>

    <!-- Content wrapper -->
    <div class="sheet-inner">

      <!-- Title -->
      {#if title}
        <div class="sheet-title-row">
          <h2 class="sheet-title">{title}</h2>
        </div>
      {/if}

      <!-- Slotted content -->
      <div class="sheet-content">
        {@render children?.()}
      </div>

    </div>

    <!-- Footer -->
    {#if footer}
      <div class="sheet-footer">
        {@render footer()}
      </div>
    {/if}

  </div>
{/if}

<style>
  /* ── Backdrop ── */
  .backdrop {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.7);
    z-index: 100;
  }

  /* ── Sheet ── */
  .sheet {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    width: 100%;
    background: #F9FAFB;
    border-radius: 16px 16px 0 0;
    z-index: 101;
    display: flex;
    flex-direction: column;
    max-height: 90vh;
    overflow: hidden;
    box-shadow: 0px -4px 24px rgba(0, 0, 0, 0.12);
  }

  /* ── Handle ── */
  .handle-wrap {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 8px 0 0;
    flex-shrink: 0;
  }

  .handle {
    width: 32px;
    height: 4px;
    background: #9E9E9E;
    border-radius: 500px;
  }

  /* ── Inner scrollable area ── */
  .sheet-inner {
    display: flex;
    flex-direction: column;
    gap: 24px;
    overflow-y: auto;
    flex: 1;
    padding: 24px 0 0;
    scrollbar-width: none;
    -ms-overflow-style: none;
  }
  .sheet-inner::-webkit-scrollbar { display: none; }

  /* ── Title ── */
  .sheet-title-row {
    padding: 0 16px;
    flex-shrink: 0;
  }

  .sheet-title {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 20px;
    line-height: 24px;
    color: #111827;
  }

  /* ── Content slot ── */
  .sheet-content {
    display: flex;
    flex-direction: column;
    padding: 0 16px 20px;
    gap: 0;
  }

  /* ── Footer slot ── */
  .sheet-footer {
    background: #F9FAFB;
    padding: 20px 16px calc(24px + env(safe-area-inset-bottom));
    flex-shrink: 0;
    display: flex;
    flex-direction: column;
    gap: 8px;
    border-top: 1px solid #F3F4F6;
  }
</style>
