<!--
  BottomSheetListItem.svelte
  ==========================
  A single list item for use inside BottomSheet content.
  Matches Figma node 80:4206 — checkbox icon + text, with optional divider.

  Props:
    text      — the item text
    checked   — whether the checkbox is checked (bindable)
    divider   — show bottom divider (default true)

  Usage:
    <BottomSheetListItem bind:checked={item.checked} text="I am above 18 years of age" />
-->
<script lang="ts">
  let {
    text = '',
    checked = $bindable(false),
    divider = true,
  }: {
    text?: string;
    checked?: boolean;
    divider?: boolean;
  } = $props();
</script>

<div class="list-item">
  <div class="item-row">
    <!-- Checkbox -->
    <button
      class="checkbox"
      class:checked
      onclick={() => checked = !checked}
      aria-label={text}
      aria-checked={checked}
      role="checkbox"
    >
      {#if checked}
        <div class="check-anim">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
            <rect width="20" height="20" rx="4" fill="#242A80"/>
            <path d="M5.5 10.2L8.2 13L14.5 6.5" stroke="#FFFFFF" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </div>
      {:else}
        <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
          <rect x="0.5" y="0.5" width="19" height="19" rx="3.5" stroke="#D1D5DB"/>
        </svg>
      {/if}
    </button>

    <!-- Text -->
    <p class="item-text">{text}</p>
  </div>

  {#if divider}
    <div class="divider"></div>
  {/if}
</div>

<style>
  .list-item {
    display: flex;
    flex-direction: column;
    background: #FFFCF4;
  }

  .item-row {
    display: flex;
    align-items: flex-start;
    gap: 8px;
    padding: 10px 12px;
  }

  .checkbox {
    width: 20px;
    height: 20px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;
    flex-shrink: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-top: 1px;
  }

  .check-anim {
    animation: checkPop 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);
    display: flex;
  }
  @keyframes checkPop {
    0%   { transform: scale(0); opacity: 0; }
    60%  { transform: scale(1.15); opacity: 1; }
    100% { transform: scale(1); }
  }

  .item-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 18px;
    color: #6B7280;
    flex: 1;
  }

  .divider {
    height: 1px;
    background: #F3F4F6;
    margin: 0 12px;
  }
</style>
