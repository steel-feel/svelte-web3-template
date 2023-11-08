<script>
  import { ethers } from "ethers";
  import { onMount } from "svelte";
  import * as  gateFiSDK from "@gatefi/js-sdk"


  let provider, wallet;
  let overlayInstanceSDK ;

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
  onMount(connectWallet);

  function handleUnlimit() {
        // if ( !overlayInstanceSDK || overlayInstanceSDK.current) {
          // const randomString = require('crypto').randomBytes(32).toString('hex');
          overlayInstanceSDK = new gateFiSDK.GateFiSDK({
            merchantId: "399e821f-a9f6-4aaa-a5ae-d01933bfeec8",
            displayMode: gateFiSDK.GateFiDisplayModeEnum.Overlay,
            nodeSelector: ".overlay-place",
            isSandbox: true,
            walletAddress: "0xb43Ae6CC2060e31790d5A7FDAAea828681a9bB4B",
            walletLock: true,
            email: "ojzvaueuanim@arxxwalls.com",
            externalId: "23234330324", //should be new for each request
            defaultFiat: {
              currency: "USD",
              amount: "33",
            },
            defaultCrypto: {
              currency: "ETH"
            },
          });
        // }
      
        // overlayInstanceSDK.current?.show();
        // setIsOverlayVisible(true);
      }
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

  <div>
    <button  on:click={handleUnlimit}>open Unlimit</button>
  </div>
  <div class="overlay-place"></div>
</main>

<style>
</style>
