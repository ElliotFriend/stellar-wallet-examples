<script>
  import { Route } from 'tinro';

  import CreateClaimableBalance from './lib/ClaimableBalance/CreateClaimableBalance.svelte';
  import ClaimClaimableBalance from './lib/ClaimableBalance/ClaimClaimableBalance.svelte';
</script>

<nav>
  <a href="/">Home</a>
  <a href="/create-cb">Create Claimable Balance</a>
  <a href="/claim-cb">Claim Claimable Balance</a>
</nav>

<main>
  <h1>Advanced Wallet Examples</h1>

  <Route path="/">
    <h2>Here are the available examples:</h2>
    <ul>
      <li><a href="/create-cb">Create Claimable Balance</a></li>
      <li><a href="/claim-cb">Claim Claimable Balance</a></li>
    </ul>
  </Route>

  <Route path="/create-cb">
    <div class="card">
      <CreateClaimableBalance />
    </div>
  </Route>

  <Route path="claim-cb">
    <div class="card">
      <ClaimClaimableBalance />
    </div>
  </Route>

</main>

<style>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #888;
  }
</style>
