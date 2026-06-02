<!--
  TextField.svelte — Floating-label text field
  Label centered when empty, floats to top on focus/value with 4px gap.
-->
<script lang="ts">
  let {
    label = '',
    value = $bindable(''),
    type = 'text',
    inputmode = undefined,
    placeholder = '',
    maxlength = undefined,
    supportText = '',
    errorText = '',
    endIcon = '',
    autocapitalize = undefined,
    disabled = false,
    autofocus = false,
    oninput = undefined,
    onblur = undefined,
  }: {
    label?: string;
    value?: string;
    type?: string;
    inputmode?: string | undefined;
    placeholder?: string;
    maxlength?: number | undefined;
    supportText?: string;
    errorText?: string;
    endIcon?: string;
    autocapitalize?: string | undefined;
    disabled?: boolean;
    autofocus?: boolean;
    oninput?: ((e: Event) => void) | undefined;
    onblur?: (() => void) | undefined;
  } = $props();

  import { onMount } from 'svelte';

  let focused = $state(false);
  let isActive = $derived(focused || value.length > 0);
  let hasError = $derived(errorText.length > 0);
  let inputEl = $state<HTMLInputElement | null>(null);

  onMount(() => {
    if (autofocus) {
      setTimeout(() => inputEl?.focus(), 400);
    }
  });

  function handleFocus() { focused = true; }
  function handleBlur() { focused = false; onblur?.(); }
  function handleInput(e: Event) { value = (e.target as HTMLInputElement).value; oninput?.(e); }
</script>

<div class="tf-group">
  <!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
  <div
    class="tf"
    class:active={isActive}
    class:focused
    class:error={hasError}
    class:disabled
    onclick={() => inputEl?.focus()}
  >
    <div class="tf-content" class:active={isActive}>
      <span class="tf-label" class:floated={isActive}>{label}</span>
      <input
        bind:this={inputEl}
        {type}
        inputmode={inputmode}
        placeholder={isActive ? placeholder : ''}
        {value}
        maxlength={maxlength}
        autocapitalize={autocapitalize}
        {disabled}
        oninput={handleInput}
        onfocus={handleFocus}
        onblur={handleBlur}
        class="tf-input"
        class:shown={isActive}
      />
    </div>

    {#if endIcon && isActive}
      <i class="ph {endIcon} tf-end-icon" style="font-size:20px"></i>
    {/if}
  </div>

  {#if hasError}
    <p class="tf-helper tf-error">{errorText}</p>
  {:else if supportText}
    <p class="tf-helper">{@html supportText}</p>
  {/if}
</div>

<style>
  .tf-group {
    display: flex;
    flex-direction: column;
    gap: 6px;
  }

  .tf {
    display: flex;
    align-items: center;
    gap: 16px;
    background: #FFFFFF;
    border: 1px solid #D1D5DB;
    border-radius: 8px;
    padding: 10px 12px;
    min-height: 52px;
    cursor: text;
    transition: border-color 0.15s ease;
  }
  .tf.focused { border-color: #242A80; }
  .tf.error   { border-color: #B91C1C; }
  .tf.disabled { opacity: 0.5; cursor: not-allowed; }

  .tf-content {
    display: flex;
    flex-direction: column;
    flex: 1;
    min-width: 0;
    justify-content: center;
  }
  .tf-content.active {
    gap: 4px;
  }

  .tf-label {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 1.5;
    color: #6B7280;
    pointer-events: none;
    transition: color 0.18s ease;
  }
  .tf-label.floated {
    color: #111827;
  }

  .tf-input {
    width: 100%;
    background: none;
    border: none;
    outline: none;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    line-height: 1.43;
    color: #111827;
    padding: 0;
    height: 0;
    opacity: 0;
    overflow: hidden;
    transition: height 0.15s ease, opacity 0.15s ease;
  }
  .tf-input.shown {
    height: 20px;
    opacity: 1;
  }
  .tf-input::placeholder {
    color: #9CA3AF;
    font-weight: 400;
  }

  .tf-end-icon { flex-shrink: 0; }

  .tf-helper {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 1.5;
    color: #6B7280;
    padding: 0 4px;
  }
  .tf-error { color: #B91C1C; }

  .tf-helper :global(a),
  .tf-helper :global(button) {
    background: none;
    border: none;
    cursor: pointer;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 12px;
    color: #242A80;
    padding: 0;
    text-decoration: none;
    display: inline;
  }
</style>
