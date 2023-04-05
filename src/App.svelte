<script>
      import {ethers} from "ethers";
      import { onMount } from "svelte";
      let provider,wallet;

      let userAddress;
      async function connectWallet() {
        //@ts-ignore
        const windowEth = window.ethereum;
        provider = new ethers.providers.Web3Provider(windowEth)

        await provider.send("eth_requestAccounts",[])
        wallet = await provider.getSigner()
        
        //user Address
        userAddress = await wallet.getAddress()
      }
      onMount(connectWallet);

</script>

<main>
 
  <h1>Web3 Svelte template</h1>

  <h2>logged in user 
    {#if userAddress }
    <span>{userAddress}</span>
    {:else}
    <span> Wallet not connected</span>
    {/if}
  
  </h2>


</main>

<style>

</style>
