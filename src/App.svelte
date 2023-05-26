<script>
  import { AuthProvider } from "@arcana/auth";

  import axios from "axios";

  import { ethers } from "ethers";
  import { onMount } from "svelte";
  import SmartAccount from "@biconomy-sdk-dev/smart-account";
  import { ChainId } from "@biconomy-sdk-dev/core-types";

import {erc20abi} from "./assets/helper.js"

  let provider, wallet;
  let userAddress;
  let greeting_value;
  let api;

  async function connectArcana() {
    const clientId = "<arcana-app-client-Id>";
    const auth = new AuthProvider(`${clientId}`, {
      //required
      position: "right", //defaults to right
      theme: "dark", //defaults to dark
      alwaysVisible: true,
    });

    await auth.init().catch((e) => {
      console.log(e);
    });

    await auth.connect().catch((e) => {
      console.log(e);
    });

    await initWallet(auth.provider);
    console.log("arcana connected");
  }

  async function initWallet(p) {
    api = axios.create({
      insecureHTTPParser: true,
      baseURL: "https://paymaster-dashboard-backend.prod.biconomy.io",
      headers: {
        Authorization: `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6Im9qenZhdWV1YW5pbUBhcnh4d2FsbHMuY29tIiwic3ViIjoiMWU3MThlN2MtN2FjYi00YjQ1LWFhOTctYzM0N2U5MjQzYTk3IiwiaWF0IjoxNjg0NzgwOTQwLCJleHAiOjE2ODczNzI5NDB9.EKEV2G5AtrXGT83cQ9Ux-Sy_B3BgptfwXZbTBUVMhj0`,
      },
    });

    //  await  p.request({ method: 'eth_requestAccounts' });

    provider = new ethers.providers.Web3Provider(p);

    await provider.send("eth_requestAccounts", []);

    console.log("connected", p.connected);
    wallet = await provider.getSigner();

    //user Address
    userAddress = await wallet.getAddress();
  }

  async function connectWallet() {
    //@ts-ignore
    const windowEth = window.ethereum;
    await initWallet(windowEth);
  }
  onMount(connectArcana);

  /* ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  ---------------------
  | Biconomy SCW Functions |
  --------------------- 
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~*/
  let smartAccount, scwAddress;
  async function loadBico() {
    // Initialize the Smart Account
    // All values are optional except networkConfig only in the case of gasless dappAPIKey is required
    let options = {
      activeNetworkId: ChainId.POLYGON_MUMBAI,
      supportedNetworksIds: [ChainId.POLYGON_MUMBAI],
      networkConfig: [
        {
          chainId: ChainId.POLYGON_MUMBAI,
          // Dapp API Key you will get from new Biconomy dashboard that will be live soon
          // Meanwhile you can use the test dapp api key mentioned above
          dappAPIKey: dAppApiKey,
          providerUrl: "https://rpc.ankr.com/polygon_mumbai",
        },
      ],
    };

    // this provider is from the social login which we created in previous setup
    smartAccount = new SmartAccount(provider, options);
    smartAccount = await smartAccount.init();

    scwAddress = smartAccount.address;
  }

  let toAddress, amount;
  async function doTx() {

    /*
    ERC20 RussCoin on Mumbai
    Address 0x3B51Fbe2bec78D888cF262C58b6Cb20b692EA97C
    Owner for minting 0xE2Dae0f0F6EE2F3b67EC1b911D59FF92e678388e
    */

    // One needs to prepare the transaction data
    // Here we will be transferring ERC 20 tokens from the Smart Contract Wallet to an address
    const Erc20Interface = new ethers.utils.Interface(erc20abi);

    // Encode an ERC-20 token transfer to recipientAddress of the specified amount
    const encodedData = Erc20Interface.encodeFunctionData("transfer", [
      toAddress, ethers.utils.parseEther(amount + "")
    ]);

    const erc20Address = "0x3B51Fbe2bec78D888cF262C58b6Cb20b692EA97C";

    // You need to create transaction objects of the following interface
    const tx1 = {
      from: scwAddress,
      to: erc20Address, // destination smart contract address
      data: encodedData,
    };

    // Optional: Transaction subscription. One can subscribe to various transaction states
    // Event listener that gets triggered once a hash is generetaed
    smartAccount.on("txHashGenerated", (response) => {
      console.log("txHashGenerated event received via emitter", response);
    });
    smartAccount.on("onHashChanged", (response) => {
      console.log("onHashChanged event received via emitter", response);
    });
    // Event listener that gets triggered once a transaction is mined
    smartAccount.on("txMined", (response) => {
      console.log("txMined event received via emitter", response);
      alert("Success: Tx done successfully");
    });
    // Event listener that gets triggered on any error
    smartAccount.on("error", (response) => {
      console.log("error event received via emitter", response);
    });

    // Sending gasless transaction
    const txResponse = await smartAccount.sendTransaction({
      transaction: tx1,
    });
    console.log("tx hash generated", txResponse.hash);
    // If you do not subscribe to listener, one can also get the receipt like shown below
    const receipt = await txResponse.wait();
    console.log("tx receipt", receipt);

    // DONE! You just sent a gasless transaction
    alert("Success: Tx submitted successfully");
  }

  /* ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  ---------------------
  | DASHBOARD Functions |
  --------------------- 
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~*/

  let appName,
    network = 80001;

  let dappId,
    dAppApiKey ;
  async function createApp() {
    let res = await api.post("/api/v1/dapps", {
      chainId: network,
      name: appName,
    });

    dappId = res.data.data.id;
    dAppApiKey = res.data.data.apiKey;

    alert(`Success: app created success fully 
          ID ${dappId}`);
  }

  async function setupGasTank() {
    //fetch funding message
    const res = await api.get("/api/v1/paymasters/funding-message");
    const fMessage = res.data.data.fundingMessage;

    let signature = await wallet.signMessage(fMessage);

    //send signature and wallet address to link dappId and paymaster gas tank owner mapping
    const resPatch = await api.patch("/api/v1/dapps", {
      address: userAddress,
      dappId,
      signature,
      type: "paymasterFundingKey",
    });

    if (resPatch.data.statusCode != 200) {
      alert("Error: unable to register gas tank request");
      return;
    }

    alert(`Success: gas tank setup `);
  }

  let appDetails;
  async function fetchApp() {
    await provider.send("eth_requestAccounts", []);
    const res = await api.get(`/api/v1/dapps/${dappId}`);
    appDetails = res.data.data;
  }

  let tankValue;
  async function GasTank() {
    const res = await api.get(`/api/v1/paymasters/default?chainId=${network}`);
    //initialise paymaster contract
    const paymasterContract = new ethers.Contract(
      res.data.data.address,
      res.data.data.abi,
      wallet
    );

    //call DepositFor function
    let tx = await paymasterContract.depositFor(userAddress, {
      value: ethers.utils.parseEther(tankValue + ""),
    });

    await tx.wait();

    alert(`Success: Gas Tank filled`);
  }
</script>

<main>
  <h1>Biconomy PoC</h1>
  <h2>Dashboard Functions</h2>
  <h5>
    logged in user
    {#if userAddress}
      <span>{userAddress}</span>
    {:else}
      <span> Wallet not connected</span>
    {/if}
  </h5>

  <div class="container con-border">
    <h5>Create App</h5>
    <div class="grid">
      <label for="appname">
        App Name
        <input
          bind:value={appName}
          type="text"
          id="appname"
          name="appname"
          placeholder="App name"
          required
        />
      </label>

      <label for="network">
        Network
        <input
          bind:value={network}
          type="number"
          id="network"
          name="network"
          placeholder="Network"
          required
        />
      </label>
    </div>

    <button role="button" class="secondary" on:click={createApp}
      >Create App</button
    >
  </div>

  <div class="container con-border">
    <h5>Setup Gas Tank</h5>
    <button role="button" class="secondary" on:click={setupGasTank}
      >Setup Gas tank</button
    >
  </div>

  <div class="container con-border">
    <h5>Fetch App details</h5>
    <button role="button" class="secondary" on:click={fetchApp}>Fetch</button>
    <div class="resp">{JSON.stringify(appDetails)}</div>
  </div>

  <div class="container con-border">
    <h5>Fill gas tank</h5>
    <div class="grid">
      <label for="tankValue">
        Tank Value in ethers
        <input
          bind:value={tankValue}
          type="number"
          id="tankValue"
          name="tankValue"
          placeholder="Tank Value"
        />
      </label>
    </div>
    <button role="button" class="secondary" on:click={GasTank}
      >Fill gas tank</button
    >
  </div>

  <h2>Smart Wallet Functions</h2>

  <div class="container con-border">
    <h5>
      Smart contract account
      {#if scwAddress}
        <span>{scwAddress}</span>
      {:else}
        <span> scw not connected</span>
      {/if}
    </h5>
    <button role="button" class="secondary" on:click={loadBico}
      >Load Biconomy SCW
    </button>
  </div>

  <div class="container con-border">
    <div class="grid">
    <label for="toAddress">To
    <input
      bind:value={toAddress}
      type="text"
      id="toAddress"
      placeholder="To Address"
    />
  </label>
    <label for="amount">Amount
    <input
      bind:value={amount}
      type="number"
      id="amount"
      placeholder="Amount"
    />
  </label>
  </div>
    <button role="button" class="secondary" on:click={doTx}>Go Gasless</button>
  </div>
</main>

<style>
  .resp {
    border: 4px solid green;
  }
  .con-border {
    margin: 1rem;
    border-style: dashed;
    border-color: red;
  }
</style>
