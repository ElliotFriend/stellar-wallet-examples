<script>
  import { writable } from 'svelte/store';

  const source = writable('GBCZUDIPBLG2A5VJSUWH2J547QZXVCTA3WMCVNGHHD3PYFALJPWO2G3J');
  const destination = writable('GDRZF5NWK5GZX6WTGFXP2RRPEXIYTSLM3DYZCV6E5NYXZTX4GEMDGAV4');
  const asset = writable({
    code: "XLM",
    issuer: null,
  });
  const amount = writable(10);
  const secret = writable('SDJICCM5LOMHMTF3ZKABLEO3V3P3XDVUSR6GCELQAUFPMP7JLQOSSUPP')
  const sourceCB = writable(false);
  const txHash = writable('');
  const cbId = writable('');

  // TODO: More than just unconditional predicates

  const createCB = async (e) => {
    e.preventDefault();
    // load the source account from the horizon server
    const keypair = StellarSdk.Keypair.fromSecret($secret);
    const server = new StellarSdk.Server('https://horizon-testnet.stellar.org');
    const account = await server.loadAccount($source);

    // create the desired claimants for this claimable balance
    const destinationClaimant = new StellarSdk.Claimant(
      $destination,
      StellarSdk.Claimant.predicateUnconditional()
    )
    const sourceClaimant = new StellarSdk.Claimant(
      $source,
      StellarSdk.Claimant.predicateUnconditional()
    )
    let claimants = [destinationClaimant];
    $sourceCB && claimants.push(sourceClaimant);

    // create the asset this claimable balance will put on hold
    const cbAsset = $asset.code === "XLM"
      ? StellarSdk.Asset.native()
      : new StellarSdk.Asset($asset.code, $asset.issuer)

    // build the transaction with a single `createClaimableBalance` operation
    const transaction = new StellarSdk.TransactionBuilder(
      account, {
        fee: StellarSdk.BASE_FEE,
        networkPassphrase: StellarSdk.Networks.TESTNET
      })
      .addOperation(StellarSdk.Operation.createClaimableBalance({
        asset: cbAsset,
        amount: $amount.toString(),
        claimants: claimants
      }))
      .setTimeout(30)
      .build()

    // sign and submit the transaction
    transaction.sign(keypair);
    try {
      let res = await server.submitTransaction(transaction);
      // console.log(`Transaction Successful! Hash: ${res.hash}`)
      txHash.set(res.hash);
      // console.log(`Claimable Balance ID: ${transaction.getClaimableBalanceId(0)}`)
      cbId.set(transaction.getClaimableBalanceId(0));
    } catch (error) {
      console.error(`${error}. More details:\n${JSON.stringify(error.response.data.extras, null, 2)}`)
    }
  }
</script>

<h2>Create Claimable Balance</h2>

<form on:submit={createCB}>
  <label for="pubkey">Source Public Key</label><br />
  <input type="text" name="pubkey" id="pubkey" bind:value={$source} /><br />

  <label for="seckey">Source Secret Key</label><br />
  <input type="password" name="seckey" id="seckey" bind:value={$secret} /><br />

  <label for="destination">Destination Address</label><br />
  <input type="text" name="destination" id="destination" bind:value={$destination} /><br />

  <input type="checkbox" name="selfClaimant" id="selfClaimant" bind:checked={$sourceCB}/>
  <label for="selfClaimant">Include source account as an unconditional claimant</label><br />

  <!-- TODO: dropdown to select assets the account actually holds? -->
  <label for="cbAsset">CB Asset (asset code, asset issuer)</label><br />
  <input type="text" name="cbAsset" id="cbAsset" bind:value={$asset.code} />
  <input type="text" name="cbIssuer" id="cbIssuer" bind:value={$asset.issuer} /><br />

  <!-- TODO: check for existing balances  -->
  <label for="cbAmount">CB Amount</label><br />
  <input type="number" name="cbAmount" id="cbAmount" bind:value={$amount} /><br />

  <button type="submit">Create CB!</button>
</form>

<p>Transaction Hash: {$txHash}</p>
<p>Claimable Balance ID: {$cbId}</p>

<style>
</style>