<script lang="ts">
  import type { Maybe, UserData } from "./types";

  export let fetchedUsername: string;
  export let fetchedAvatar: string;
  export let data: Maybe<UserData>;
</script>

<div id="gh-user-data">
  <img src={fetchedAvatar} alt="User avatar" />
  <div id="gh-user-main-data">
    <h2>
      <a href={`https://github.com/${fetchedUsername}`} target="_blank">
        {fetchedUsername}
      </a>
    </h2>
    {#if data}
      <p>
        <strong>Name:</strong>
        {data.name}
      </p>
      <p>
        <strong>Location:</strong>
        {data.location}
      </p>
      <p>
        <strong>Followers:</strong>
        {data.followers}
      </p>
      <p>
        <strong>Following:</strong>
        {data.following}
      </p>
      <p>
        <strong>Public Repos:</strong>
        {data.public_repos}
      </p>
    {/if}
  </div>
</div>
