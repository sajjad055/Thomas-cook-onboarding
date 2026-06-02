<script lang="ts">
  let { percent = 10, size = 48 }: { percent?: number; size?: number } = $props();

  const r = (size - 6) / 2;
  const cx = size / 2;
  const cy = size / 2;
  const circumference = 2 * Math.PI * r;

  let dashOffset = $derived(circumference - (percent / 100) * circumference);

  const gradientId = `prog-${Math.random().toString(36).slice(2)}`;
</script>

<div class="ring-wrap" style="width:{size}px;height:{size}px">
  <svg width={size} height={size} viewBox="0 0 {size} {size}">
    <defs>
      <linearGradient id={gradientId} x1="0%" y1="0%" x2="100%" y2="100%">
        <stop offset="0%" stop-color="#4ADE80"/>
        <stop offset="100%" stop-color="#15803D"/>
      </linearGradient>
    </defs>
    <!-- Track -->
    <circle
      cx={cx} cy={cy} r={r}
      fill="none"
      stroke="#E5E7EB"
      stroke-width="5"
    />
    <!-- Progress -->
    <circle
      cx={cx} cy={cy} r={r}
      fill="none"
      stroke="url(#{gradientId})"
      stroke-width="5"
      stroke-linecap="round"
      stroke-dasharray={circumference}
      stroke-dashoffset={dashOffset}
      transform="rotate(-90 {cx} {cy})"
      style="transition: stroke-dashoffset 0.4s ease"
    />
  </svg>
  <span class="label">{percent}%</span>
</div>

<style>
  .ring-wrap {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }
  .label {
    position: absolute;
    font-family: 'Nunito Sans', sans-serif;
    font-weight: 600;
    font-size: 10px;
    line-height: 1;
    color: #111827;
  }
</style>
