<script lang="ts">
  import type { RateLimitData } from "./types";

  export let rateLimitData: RateLimitData;
  export let cost: number;
</script>

<div class="rate-limit-section">
  <p>
    Cost per search: {cost}.
  </p>
  <p>
    Available searches: {rateLimitData.rate.remaining} / {rateLimitData.rate
      .limit}.
  </p>
  <p>
    Rate limit resets at: {new Date(
      rateLimitData.rate.reset * 1000,
    ).toLocaleTimeString()}
  </p>
  {#if rateLimitData.rate.remaining === 0}
    <p>Rate limit exceeded. Please try again later.</p>
  {/if}
</div>
