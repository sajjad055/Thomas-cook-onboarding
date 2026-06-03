<script lang="ts">
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';
  import { browser } from '$app/environment';

  const messages = [
    { text: 'Pulling up your details', sub: 'This will only take a moment' },
    { text: 'Checking your credit profile', sub: 'Your data is fully encrypted' },
    { text: 'Reviewing income details', sub: 'Almost done' },
    { text: 'Looking good so far', sub: 'Just a few more checks' },
    { text: 'Wrapping things up', sub: 'Your offer is being prepared' },
  ];

  let messageIndex = $state(0);
  let done = $state(false);
  let lottieReady = $state(false);

  onMount(async () => {
    if (browser) {
      try {
        await import('@lottiefiles/dotlottie-wc');
      } catch (e) {
        console.warn('Lottie load failed', e);
      }
    }
    lottieReady = true;

    const interval = setInterval(() => {
      if (messageIndex < messages.length - 1) {
        messageIndex++;
      }
    }, 3600);

    setTimeout(() => {
      clearInterval(interval);
      done = true;
      setTimeout(() => { goto(`${base}/card-name`); }, 2500);
    }, 18000);

    return () => clearInterval(interval);
  });
</script>

<svelte:head><title>Verifying – IOB</title></svelte:head>

<div class="screen">

  {#if !done}
    <!-- Rocket loader -->
    <div class="longfazers">
      <span></span>
      <span></span>
      <span></span>
      <span></span>
    </div>

    <div class="loader">
      <span>
        <span></span>
        <span></span>
        <span></span>
        <span></span>
      </span>
      <div class="base">
        <span></span>
        <div class="face"></div>
      </div>
    </div>
  {:else}
    {#if lottieReady}
      <div class="lottie-wrap" in:fade={{ duration: 400 }}>
        <dotlottie-wc
          src="{base}/success.lottie"
          autoplay
          speed="0.7"
          style="width: 120px; height: 120px;"
        ></dotlottie-wc>
      </div>
    {/if}
  {/if}

  <!-- Message -->
  <div class="message-block">
    {#key done ? 'done' : messageIndex}
      <p class="main-text" in:fade={{ duration: 500, delay: 80 }}>
        {#if done}
          You are all set
        {:else}
          {messages[messageIndex].text}
        {/if}
      </p>
      <p class="sub-text" in:fade={{ duration: 500, delay: 200 }}>
        {#if done}
          Redirecting you now
        {:else}
          {messages[messageIndex].sub}
        {/if}
      </p>
    {/key}
  </div>

</div>

<style>
  .screen {
    width: 100%;
    min-height: 100dvh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: #FFFCF4;
    padding: 32px 24px;
    position: relative;
    overflow: hidden;
  }

  /* Rocket loader */
  .loader {
    position: relative;
    animation: speeder 0.4s linear infinite;
    margin-bottom: 40px;
  }

  .loader > span {
    height: 5px;
    width: 35px;
    background: #184595;
    position: absolute;
    top: -19px;
    left: 60px;
    border-radius: 2px 10px 1px 0;
  }

  .base span {
    position: absolute;
    width: 0;
    height: 0;
    border-top: 6px solid transparent;
    border-right: 100px solid #184595;
    border-bottom: 6px solid transparent;
  }

  .base span:before {
    content: "";
    height: 22px;
    width: 22px;
    border-radius: 50%;
    background: #184595;
    position: absolute;
    right: -110px;
    top: -16px;
  }

  .base span:after {
    content: "";
    position: absolute;
    width: 0;
    height: 0;
    border-top: 0 solid transparent;
    border-right: 55px solid #184595;
    border-bottom: 16px solid transparent;
    top: -16px;
    right: -98px;
  }

  .face {
    position: absolute;
    height: 12px;
    width: 20px;
    background: #184595;
    border-radius: 20px 20px 0 0;
    transform: rotate(-40deg);
    right: -125px;
    top: -15px;
  }

  .face:after {
    content: "";
    height: 12px;
    width: 12px;
    background: #184595;
    right: 4px;
    top: 7px;
    position: absolute;
    transform: rotate(40deg);
    transform-origin: 50% 50%;
    border-radius: 0 0 0 2px;
  }

  .loader > span > span:nth-child(1),
  .loader > span > span:nth-child(2),
  .loader > span > span:nth-child(3),
  .loader > span > span:nth-child(4) {
    width: 30px;
    height: 1px;
    background: #184595;
    position: absolute;
    animation: fazer1 0.2s linear infinite;
  }

  .loader > span > span:nth-child(2) {
    top: 3px;
    animation: fazer2 0.4s linear infinite;
  }

  .loader > span > span:nth-child(3) {
    top: 1px;
    animation: fazer3 0.4s linear infinite;
    animation-delay: -1s;
  }

  .loader > span > span:nth-child(4) {
    top: 4px;
    animation: fazer4 1s linear infinite;
    animation-delay: -1s;
  }

  @keyframes fazer1 {
    0% { left: 0; }
    100% { left: -80px; opacity: 0; }
  }
  @keyframes fazer2 {
    0% { left: 0; }
    100% { left: -100px; opacity: 0; }
  }
  @keyframes fazer3 {
    0% { left: 0; }
    100% { left: -50px; opacity: 0; }
  }
  @keyframes fazer4 {
    0% { left: 0; }
    100% { left: -150px; opacity: 0; }
  }

  @keyframes speeder {
    0% { transform: translate(2px, 1px) rotate(0deg); }
    10% { transform: translate(-1px, -3px) rotate(-1deg); }
    20% { transform: translate(-2px, 0px) rotate(1deg); }
    30% { transform: translate(1px, 2px) rotate(0deg); }
    40% { transform: translate(1px, -1px) rotate(1deg); }
    50% { transform: translate(-1px, 3px) rotate(-1deg); }
    60% { transform: translate(-1px, 1px) rotate(0deg); }
    70% { transform: translate(3px, 1px) rotate(-1deg); }
    80% { transform: translate(-2px, -1px) rotate(1deg); }
    90% { transform: translate(2px, 1px) rotate(0deg); }
    100% { transform: translate(1px, -2px) rotate(-1deg); }
  }

  /* Long fazers (background streaks) */
  .longfazers {
    position: absolute;
    width: 100%;
    height: 100%;
  }

  .longfazers span {
    position: absolute;
    height: 2px;
    width: 20%;
    background: rgba(24, 69, 149, 0.15);
  }

  .longfazers span:nth-child(1) {
    top: 20%;
    animation: lf 0.6s linear infinite;
    animation-delay: -5s;
  }
  .longfazers span:nth-child(2) {
    top: 40%;
    animation: lf2 0.8s linear infinite;
    animation-delay: -1s;
  }
  .longfazers span:nth-child(3) {
    top: 60%;
    animation: lf3 0.6s linear infinite;
  }
  .longfazers span:nth-child(4) {
    top: 80%;
    animation: lf4 0.5s linear infinite;
    animation-delay: -3s;
  }

  @keyframes lf {
    0% { left: 200%; }
    100% { left: -200%; opacity: 0; }
  }
  @keyframes lf2 {
    0% { left: 200%; }
    100% { left: -200%; opacity: 0; }
  }
  @keyframes lf3 {
    0% { left: 200%; }
    100% { left: -100%; opacity: 0; }
  }
  @keyframes lf4 {
    0% { left: 200%; }
    100% { left: -100%; opacity: 0; }
  }

  /* Success lottie */
  .lottie-wrap {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-bottom: 40px;
  }

  /* Text */
  .message-block {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 8px;
    min-height: 60px;
    position: relative;
    z-index: 1;
  }

  .main-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 20px;
    color: #111827;
    text-align: center;
    line-height: 1.3;
  }

  .sub-text {
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 400;
    font-size: 14px;
    color: #6B7280;
    text-align: center;
    line-height: 1.5;
  }
</style>
