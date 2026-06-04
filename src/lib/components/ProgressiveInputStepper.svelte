<!--
  ProgressiveStepper.svelte — Vertical progressive stepper
  Reveals one step at a time. Next step appears after the current is answered.
  Supports chip selection and text input step types.

  Props:
    - steps: Array<{ label: string; type?: 'chips' | 'text'; options?: string[]; placeholder?: string; value: string }>
    - onchange: (stepIndex: number, value: string) => void
-->
<script lang="ts">
  import { slide } from 'svelte/transition';

  let {
    steps = [],
    onchange = () => {},
    mode = 'dependent',
  }: {
    steps: { label: string; type?: 'chips' | 'text'; options?: string[]; placeholder?: string; value: string }[];
    onchange?: (stepIndex: number, value: string) => void;
    mode?: 'dependent' | 'independent';
  } = $props();

  // Which step is currently active (expanded)
  let activeStep = $derived(steps.findIndex(s => s.value === ''));

  let textInputValue = $state('');

  function selectOption(stepIndex: number, option: string) {
    onchange(stepIndex, option);
  }

  function submitText(stepIndex: number) {
    if (textInputValue.trim().length === 0) return;
    onchange(stepIndex, textInputValue.trim());
    textInputValue = '';
  }

  function handleTextKeydown(stepIndex: number, e: KeyboardEvent) {
    if (e.key === 'Enter') {
      submitText(stepIndex);
    }
  }

  function editStep(stepIndex: number) {
    if (mode === 'dependent') {
      // Clear this step and all subsequent
      for (let i = stepIndex; i < steps.length; i++) {
        onchange(i, '');
      }
      textInputValue = '';
    } else {
      // Only clear the edited step
      onchange(stepIndex, '');
      textInputValue = '';
    }
  }
</script>

<div class="stepper-wrapper">
  <div class="stepper">
  {#each steps as step, i}
    {@const isCompleted = step.value !== ''}
    {@const isActive = i === activeStep}
    {@const isLocked = i > activeStep && activeStep !== -1}
    {@const isVisible = mode === 'independent' || i <= (activeStep === -1 ? steps.length - 1 : activeStep) || isCompleted}

    {#if isVisible}
      <div class="step" class:completed={isCompleted} class:active={isActive} class:locked={isLocked}>
        <!-- Step indicator + connector -->
        <div class="step-left">
          <div class="step-circle" class:completed={isCompleted} class:active={isActive}>
            {#if isCompleted}
              <svg width="14" height="14" viewBox="0 0 14 14" fill="none">
                <path d="M3 7.2L5.8 10L11 4" stroke="#FFFFFF" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            {:else}
              <span class="step-number">{i + 1}</span>
            {/if}
          </div>
          {#if i < steps.length - 1 && (isCompleted || isActive)}
            <div class="step-connector" class:completed={isCompleted}></div>
          {/if}
        </div>

        <!-- Step content -->
        <div class="step-content">
          <div class="step-header">
            <span class="step-label" class:completed={isCompleted}>{step.label}</span>
            {#if isCompleted}
              <button class="step-edit" onclick={() => editStep(i)} aria-label="Edit">
                <i class="ph ph-pencil-simple" style="font-size:16px; color:#111827"></i>
              </button>
            {/if}
          </div>

          {#if isCompleted}
            <p class="step-answer">{step.value}</p>
          {/if}

          {#if isActive}
            <div class="step-options" transition:slide={{ duration: 250 }}>
              {#if (step.type || 'chips') === 'chips'}
                {#each step.options || [] as option}
                  <button
                    class="option-chip"
                    onclick={() => selectOption(i, option)}
                  >
                    {option}
                  </button>
                {/each}
              {:else}
                <div class="text-input-row">
                  <input
                    type="text"
                    class="step-text-input"
                    placeholder={step.placeholder || 'Type here...'}
                    bind:value={textInputValue}
                    onkeydown={(e) => handleTextKeydown(i, e)}
                  />
                  <button
                    class="text-submit-btn"
                    disabled={textInputValue.trim().length === 0}
                    onclick={() => submitText(i)}
                  >
                    <i class="ph ph-arrow-right" style="font-size:18px; color:#FFFFFF"></i>
                  </button>
                </div>
              {/if}
            </div>
          {/if}
        </div>
      </div>
    {/if}
  {/each}
</div>
</div>

<style>
  .stepper-wrapper {
    background: #FFFFFF;
    border-radius: 12px;
    padding: 16px 16px;
  }

  .stepper {
    display: flex;
    flex-direction: column;
    gap: 0;
  }

  .step {
    display: flex;
    gap: 12px;
    padding-bottom: 20px;
  }
  .step:last-child {
    padding-bottom: 0;
  }

  /* Left column: circle + connector */
  .step-left {
    display: flex;
    flex-direction: column;
    align-items: center;
    flex-shrink: 0;
    width: 28px;
  }

  .step-circle {
    width: 28px;
    height: 28px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #E5E7EB;
    flex-shrink: 0;
    transition: background 0.2s ease;
  }
  .step-circle.active {
    background: #184595;
  }
  .step-circle.completed {
    background: #15803D;
  }

  .step-number {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 700;
    font-size: 12px;
    color: #6B7280;
  }
  .step-circle.active .step-number {
    color: #FFFFFF;
  }

  .step-connector {
    width: 2px;
    flex: 1;
    min-height: 16px;
    background: #E5E7EB;
    margin-top: 4px;
    border-radius: 1px;
    transition: background 0.2s ease;
  }
  .step-connector.completed {
    background: #15803D;
  }

  /* Right column: content */
  .step-content {
    flex: 1;
    display: flex;
    flex-direction: column;
    gap: 8px;
    padding-top: 3px;
  }

  .step-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 8px;
  }

  .step-label {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    color: #111827;
    line-height: 1.4;
  }
  .step-label.completed {
    color: #6B7280;
    font-weight: 500;
  }

  .step-edit {
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;
    width: 24px;
    height: 24px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .step-answer {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    color: #111827;
    line-height: 1.4;
  }

  /* Options chips */
  .step-options {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    padding-top: 4px;
  }

  .option-chip {
    padding: 8px 16px;
    background: #FFFFFF;
    border: 1px solid #D1D5DB;
    border-radius: 20px;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 500;
    font-size: 13px;
    color: #111827;
    cursor: pointer;
    transition: background 0.15s ease, border-color 0.15s ease;
  }
  .option-chip:active {
    background: #F3F4F6;
    border-color: #184595;
  }

  /* Text input step */
  .text-input-row {
    display: flex;
    gap: 8px;
    align-items: center;
    width: 100%;
  }

  .step-text-input {
    flex: 1;
    padding: 14px 12px;
    background: #FFFFFF;
    border: 1px solid #D1D5DB;
    border-radius: 8px;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 14px;
    color: #111827;
    outline: none;
    min-height: 52px;
    transition: border-color 0.15s ease;
  }
  .step-text-input:focus {
    border-color: #D1D5DB;
  }
  .step-text-input::placeholder {
    color: #9CA3AF;
    font-weight: 400;
  }

  .text-submit-btn {
    width: 36px;
    height: 36px;
    border-radius: 50%;
    background: #184595;
    border: none;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    transition: opacity 0.15s ease;
  }
  .text-submit-btn:disabled {
    background: #D1D5DB;
    cursor: not-allowed;
  }
</style>
