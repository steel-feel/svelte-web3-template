<script>
  import { ethers } from "ethers";
  import { onMount } from "svelte";

  import { SchemaRegistry, EAS } from "@ethereum-attestation-service/eas-sdk";

  let provider, wallet;

  //Mumbai
  const schemaRegAddress = "0x55D26f9ae0203EF95494AE4C170eD35f4Cf77797",
    EASContractAddress = "0xaEF4103A04090071165F78D45D83A0C0782c2B2a",
    schemaUID =
      "0xf60c347a82870ddbe5d856aa123cf761d3611395efe59f0ce882049ba4305f41";

  let userAddress;
  async function connectWallet() {
    //@ts-ignore
    const windowEth = window.ethereum;
    provider = new ethers.providers.Web3Provider(windowEth);

    await provider.send("eth_requestAccounts", []);
    wallet = await provider.getSigner();

    //user Address
    userAddress = await wallet.getAddress();
  }

  async function fetchAttestation() {
    const eas = new EAS(EASContractAddress);
    eas.connect(provider);

    const uid ="0x9799aed9338a683edf501a4bcc20ad1d442f0a8214e06cb4b783f13cdf94d58d";

    const attestation = await eas.getAttestation(uid);

    console.log(attestation);
  }
  async function fetchSchema() {
    const schemaRegistry = new SchemaRegistry(schemaRegAddress);
    schemaRegistry.connect(provider);

    const schemaRecord = await schemaRegistry.getSchema({ uid: schemaUID });

    console.log(schemaRecord);

    // Example Output
    /*
{
  uid: '0xYourSchemaUID',
  schema: 'bytes32 proposalId, bool vote',
  resolver: '0xResolverAddress',
  revocable: true
}
*/
  }

  onMount(connectWallet);
</script>

<main>
  <h1>Web3 Svelte template</h1>

  <h2>
    logged in user
    {#if userAddress}
      <span>{userAddress}</span>
    {:else}
      <span> Wallet not connected</span>
    {/if}
  </h2>

  <button on:click={fetchSchema}>Get Schema</button>
  <button on:click={fetchAttestation}>Get Attestation</button>
</main>

<style>
</style>
