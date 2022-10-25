<script>
  import { writable } from 'svelte/store';

  const claimant = writable('GDRZF5NWK5GZX6WTGFXP2RRPEXIYTSLM3DYZCV6E5NYXZTX4GEMDGAV4');
  const secret = writable('SCHJCCPM6NKIWHEDUIWQIFUCLJJSDMW7FZ5L4SYZFSNNT2TM2WG5PSNL');
  const cbArray = writable([]);
  const selectedCBs = writable([]);
  const fetchedCBs = writable(false);
  const txHash = writable('');

  // fetch claimable balances that are available to claim
  const fetchClaimableBalances = async () => {
    const server = new StellarSdk.Server('https://horizon-testnet.stellar.org');
    const cbRecords = await server
      .claimableBalances()
      .claimant($claimant)
      .limit(200)
      .call()
      .then((res) => res.records)
    cbArray.set(cbRecords);
    fetchedCBs.set(true);
  }

  const resetState = () => {
    cbArray.set([]);
    fetchedCBs.set(false);
    txHash.set('');
  }

  // TODO: handle more than just native() asset?
  // TODO: do some checking for if balances are available or not yet

  // claim the selected claimable balances
  const claimSelectedBalances = async () => {
    // load the source account from the horizon server
    const keypair = StellarSdk.Keypair.fromSecret($secret);
    const server = new StellarSdk.Server('https://horizon-testnet.stellar.org');
    const account = await server.loadAccount($claimant);

    // begin the transaction
    let transaction = new StellarSdk.TransactionBuilder(
      account, {
        fee: StellarSdk.BASE_FEE,
        networkPassphrase: StellarSdk.Networks.TESTNET
      });

    // add a claim operation for each selected balance
    $selectedCBs.forEach(cbId => {
      transaction.addOperation(StellarSdk.Operation.claimClaimableBalance({
        balanceId: cbId
      }))
    })

    // build, sign, submit the transaction
    transaction = transaction.setTimeout(30).build()
    transaction.sign(keypair)
    try {
      let res = await server.submitTransaction(transaction);
      txHash.set(res.hash);
    } catch (error) {
      console.error(`${error}. More details:\n${JSON.stringify(error.response.data.extras, null, 2)}`)
    }
  }
</script>

<h2>Claim Claimable Balance</h2>

<form on:submit={e => e.preventDefault()}>
  <label for="claimantPubkey">Claimant Public Key</label><br />
  <input type="text" name="claimantPubkey" id="claimantPubkey" bind:value={$claimant} /><br />
  {#if !$fetchedCBs}
    <button on:click={fetchClaimableBalances}>Fetch Available CBs</button>
  {:else}
    {#if $cbArray.length !== 0}
      {#each $cbArray as record}
        <label>
          <input type="checkbox" bind:group={$selectedCBs} value={record.id}>
          {`${parseFloat(record.amount)} ${record.asset === "native" ? "XLM" : record.asset}`}
        </label><br />
      {/each}
      <label for="claimantSecret">Claimant Secret Key</label><br />
      <input type="password" name="claimantSecret" id="claimantSecret" bind:value={$secret} /><br />
      <button type="submit" on:click={claimSelectedBalances}>Claim Selected Balances</button><br />
    {:else}
      No claimable balances found<br />
    {/if}
    <button on:click={resetState}>Reset</button>
  {/if}
</form>

<p>Transaction Hash: {$txHash}</p>

<style>
</style>