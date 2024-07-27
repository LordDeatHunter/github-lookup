<script lang="ts">
  import type { UserData } from "./types";

  let username: string = "";
  let data = undefined as UserData | undefined;
  let emails = new Set<string>();
  let loading = false;
  let rateLimitData = undefined as
    | { limit: number; used: number; remaining: number; reset: number }
    | undefined;

  export const checkRateLimit = async () =>
    await fetch("https://api.github.com/rate_limit")
      .then((response) => response.json())
      .then((response) => {
        rateLimitData = response.resources.core;
      })
      .catch((error) => {
        console.error(error);
      });

  checkRateLimit();

  export const userLookup = async () => {
    await checkRateLimit();
    if (!rateLimitData || rateLimitData.remaining === 0) {
      return;
    }
    console.log(username);
    data = undefined;
    loading = true;
    emails = new Set<string>();
    // const response = await fetch(`https://api.github.com/users/${username}`);
    // const responseData = await response.json();
    let page = 1;
    const repoCommits = [];
    const reposResponse = await fetch(
      `https://api.github.com/users/${username}/repos?page=${page}&per_page=100`,
    );
    const reposData = await reposResponse.json();
    if (reposData.length === 0) {
      await checkRateLimit();
      return;
    }
    repoCommits.push(...reposData.map((repo: any) => repo.commits_url));
    const repoCommitsUrl = repoCommits[0];
    const repoCommitsResponse = await fetch(
      `${repoCommitsUrl.replace("{/sha}", "")}?author=${username}`,
    );
    const repoCommitsData = await repoCommitsResponse.json();
    console.log(repoCommitsData);
    emails = new Set<string>(
      repoCommitsData
        .map((commit: any) => commit.commit.author.email)
        .filter((email: string) => email),
    );

    // data = responseData;
    loading = false;

    await checkRateLimit();
  };
</script>

<main>
  <h1 id="gh-title">GitHub User Lookup</h1>
  <div id="gh-user-lookup">
    <div class="input-section">
      <input
        type="text"
        id="gh-user-search"
        placeholder="Type a GitHub username"
        bind:value={username}
      />
      <button id="gh-user-lookup-button" on:click={userLookup}> Lookup </button>
    </div>
    {#if loading}
      <h3>Loading...</h3>
    {/if}
    {#if rateLimitData}
      <p>
        Available searches: {rateLimitData.remaining} / {rateLimitData.limit}
      </p>
      <p>
        Rate limit resets at: {new Date(
          rateLimitData.reset * 1000,
        ).toLocaleTimeString()}
      </p>
      {#if rateLimitData.remaining === 0}
        <p>Rate limit exceeded. Please try again later.</p>
      {/if}
    {/if}
    {#if emails.size > 0}
      <div id="gh-user-emails">
        {#if emails.size > 0}
          <h2>Emails</h2>
          <ul class="no-deco-list">
            {#each emails as email}
              <li>{email}</li>
            {/each}
          </ul>
        {/if}
      </div>
    {/if}
  </div>
</main>
