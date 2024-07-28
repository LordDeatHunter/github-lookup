<script lang="ts">
  import type { Maybe, RateLimitData, ReposData, UserData } from "./types";
  import EmailSection from "./EmailSection.svelte";
  import UserDataSection from "./UserDataSection.svelte";
  import RateLimitSection from "./RateLimitSection.svelte";

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
  let invalidUser = false;
  $: cost = fetchData ? 3 : 2;

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

  export const fetchRequest = async (requestFunction: () => Promise<any>) => {
    await checkRateLimit();
    loading = true;
    if (!rateLimitData || rateLimitData.resources.core.remaining === 0) {
      return;
    }
    await requestFunction();
    await checkRateLimit();
    loading = false;
  };

  export const userLookup = async () => {
    invalidUser = false;
    data = undefined;
    emails = undefined;
    if (fetchData) {
      const response = await fetch(`https://api.github.com/users/${username}`);
      if (response.status !== 200) {
        invalidUser = true;
        return;
      }
      data = await response.json();
    }
    let page = 1;
    const repoCommits = [];
    const reposResponse = await fetch(
      `https://api.github.com/users/${username}/repos?page=${page}&per_page=100`,
    );
    if (reposResponse.status !== 200) {
      invalidUser = true;
      return;
    }
    reposData = await reposResponse.json();
    if (!reposData || reposData.length === 0) {
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
        <button
          id="gh-user-lookup-button"
          on:click={() => fetchRequest(userLookup)}
        >
          Lookup
        </button>
      </div>
      <div>
        <input type="checkbox" id="gh-data-checkbox" bind:checked={fetchData} />
        <label for="gh-data-checkbox">Show regular data</label>
      </div>
    </div>
    {#if rateLimitData != null}
      <RateLimitSection {rateLimitData} {cost} />
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
    {#if invalidUser}
      <h2 class="error-message">Invalid user.</h2>
    {/if}
  </div>
</main>
<footer>
  <p>
    Source code available on
    <a href="https://github.com/LordDeatHunter/github-lookup" target="_blank">
      GitHub
    </a>
  </p>
</footer>
