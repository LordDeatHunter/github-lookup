<script lang="ts">
  import type { Maybe, RateLimitData, ReposData, UserData } from "./types";
  import EmailSection from "./EmailSection.svelte";
  import UserDataSection from "./UserDataSection.svelte";

  let username: string = "";
  let data: Maybe<UserData>;
  let emails: Maybe<Set<string>>;
  let loading = false;
  let rateLimitData: Maybe<RateLimitData>;
  let reposData: Maybe<ReposData[]>;
  let fetchData = false;
  $: fetchedUsername = data?.login || reposData?.[0]?.owner?.login || undefined;
  $: document.title = fetchedUsername
    ? `GitHub User Lookup: ${fetchedUsername}`
    : "GitHub User Lookup";
  $: fetchedAvatar =
    data?.avatar_url || reposData?.[0]?.owner?.avatar_url || undefined;

  export const checkRateLimit = async () =>
    await fetch("https://api.github.com/rate_limit")
      .then((response) => response.json() as Promise<RateLimitData>)
      .then((response) => {
        rateLimitData = response;
      })
      .catch((error) => {
        console.error(error);
      });

  checkRateLimit();

  export const userLookup = async () => {
    await checkRateLimit();
    if (!rateLimitData || rateLimitData.resources.core.remaining === 0) {
      return;
    }
    console.log(username);
    data = undefined;
    loading = true;
    emails = new Set<string>();
    if (fetchData) {
      const response = await fetch(`https://api.github.com/users/${username}`);
      data = await response.json();
    }
    let page = 1;
    const repoCommits = [];
    const reposResponse = await fetch(
      `https://api.github.com/users/${username}/repos?page=${page}&per_page=100`,
    );
    reposData = await reposResponse.json();
    if (!reposData || reposData.length === 0) {
      loading = false;
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

    loading = false;

    await checkRateLimit();
  };
</script>

<main>
  <h1 id="gh-title">GitHub User Lookup</h1>
  <div id="gh-user-lookup">
    <div class="input-section">
      <div class="search-section">
        <input
          type="text"
          id="gh-user-search"
          placeholder="Enter a GitHub username"
          bind:value={username}
        />
        <button id="gh-user-lookup-button" on:click={userLookup}>Lookup</button>
      </div>
      <div>
        <input type="checkbox" id="gh-data-checkbox" bind:checked={fetchData} />
        <label for="gh-data-checkbox">Show regular data</label>
      </div>
    </div>
    {#if rateLimitData != null}
      <p>
        Available searches: {rateLimitData.rate.remaining} / {rateLimitData.rate
          .limit}
      </p>
      <p>
        Rate limit resets at: {new Date(
          rateLimitData.rate.reset * 1000,
        ).toLocaleTimeString()}
      </p>
      {#if rateLimitData.rate.remaining === 0}
        <p>Rate limit exceeded. Please try again later.</p>
      {/if}
    {/if}
    {#if loading}
      <span class="loader"></span>
    {:else}
      {#if fetchedUsername && fetchedAvatar}
        <UserDataSection {fetchedUsername} {fetchedAvatar} {data} />
      {/if}
      {#if emails}
        <EmailSection {emails} />
      {/if}
    {/if}
  </div>
</main>
