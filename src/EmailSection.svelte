<script lang="ts">
  export let emails: Set<string>;
</script>

<div id="gh-user-emails">
  {#if emails.size > 0}
    <h2>Emails</h2>
    <ul class="no-deco-list">
      {#each emails as email}
        <li>{email}</li>
      {/each}
    </ul>
  {:else}
    <p>No emails found.</p>
  {/if}
</div>
