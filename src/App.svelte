<script lang="ts">
  import {
    parseEther,
    BrowserProvider,
    Interface,
    Contract,
    ethers,
  } from "ethers";
  import { onMount } from "svelte";

  import { SCW, StorageType } from "@arcana/scw"; //From npm

  import { AuthProvider } from "@arcana/auth"; //From npm

 import {sessionTestAbi, erc20abi} from './utils'

  let provider, wallet;

  let userAddress;

  async function connectWallet() {
    //@ts-ignore
    const windowEth = window.ethereum;
    provider = new BrowserProvider(windowEth);

    await provider.send("eth_requestAccounts", []);
    wallet = await provider.getSigner();

    //user Address
    userAddress = await wallet.getAddress();
  }
  onMount(arcanaWallet);
  let auth;
  let scWallet: SCW;
  
  const LINK_ARB_SEPOLIA = "0xb1D4538B4571d411F07960EF2838Ce337FE1E80E";
  const LINK_BSC_TESTNET = "0x84b9B910527Ad5C03A9Ca831909E21e236EA7b06";
  const USDC_ARB_SEPOLIA = "0x75faf114eafb1BDbe2F0316DF893fd58CE46AA4d";
  const USDC_BSC_TESTNET = "0x64544969ed7EBf5f083679233325356EbE738930";
  const XNYT_ARB_SEPOLIA = "0xDC814506A7ed9BEe200c845fb04ECdAAb0ea2811";

  // let arcana_app_id = "xar_live_2f1b0f49b5682f9cab5512bf51d022f25723518a";
  // let arcana_app_id = "xar_live_e553c5570f9c4768a2656da70ecc6fd4747e7214";
  // let arcana_app_id = "xar_test_7c27043e6263eff62c6b3a348d613f5b6c9f2527";
  // let arcana_app_id = "xar_live_10df430d374e1e9505615958f9965b7fbeb894d7";
  let arcana_app_id = "xar_dev_1e3ee6a5cecc593d0dac2e1893dbe7534a174ac4"; // Arbitrum sepolia bico paymaster
  //  let arcana_app_id = "xar_test_92dfb47e769cd6304669fca1877963750b093965"; // Arbitrum sepolia bico paymaster
  /// ~~~~~~~ Arcana Wallet ~~~~~~~~~

  async function arcanaWallet() {
    auth = new AuthProvider(
      arcana_app_id, // App client ID
      {
        setWindowProvider: true, // default: false, window.ethereum not set
        connectOptions: {
          compact: false, // default: false, regular plug-and-play login UI
        },
      },
    );

    auth = await auth.init();
  }

  async function connectArcana() {
    const arcanaProvider = await auth.connect();
    provider = new BrowserProvider(arcanaProvider);
    wallet = await provider.getSigner();

    userAddress = await wallet.getAddress();
    //@ts-ignore
    window.w = wallet as SCW;
  }

  async function sendTx() {
    const tx = await wallet.sendTransaction({
      to: "0x7a8713E21e7434dC5441Fb666D252D13F380a97d",
      value: parseEther("0.00001"),
    });

    console.log({ tx });
  }

  /// ~~~~~~  Arcana Gasless ~~~~~~~~

  async function initGasLess() {
    scWallet = new SCW();
    //@ts-ignore
    console.log("window.arcana.provider", window.arcana.provider);
    //@ts-ignore
    await scWallet.init(arcana_app_id, window.arcana.provider, undefined, 0);
    scwAddress = await scWallet.getSCWAddress();
    //@ts-ignore
    window.scwi = scWallet;
    console.log("Address: " + scWallet.getSCWAddress());
  }

  async function sendGaslessTx() {
    let amount = inputValue;

    // const erc20Address = "0x06A0F0fa38AE42b7B3C8698e987862AfA58e90D9";
    const erc20Address = getErc20Contract(scWallet.chain_id);
    const toAddress = "0x7a8713E21e7434dC5441Fb666D252D13F380a97d";
    const Erc20Interface = new Interface(erc20abi);

    const encodedData = Erc20Interface.encodeFunctionData("transfer", [
      toAddress,
      amount,
    ]);

    // You need to create transaction objects of the following interface
    const tx1 = {
      from: scWallet.getSCWAddress(),
      to: erc20Address, // destination smart contract address
      data: encodedData,
    };

    // for (let i = 0; i < 5; i++) {
    let tx = await scWallet.doTx(tx1);
    await tx.wait();
    console.log(`Transfer done ${tx.userOpHash}`);
    // }
  }

  /// ~~~~~~~~~~ Biconomy gasless ~~~~~~
  let scwAddress = "";

  // type CreateSessionParam = {
  //   contractAddress: string;
  //   functionSelector: string;
  //   validUntil?: number;
  //   validAfter?: number;
  //   valueLimit?: number;
  // };
  // ~~~~~ Session management ~~~~~~
  let sess;
  async function initSession() {
    
    sess = scWallet.initSession(StorageType.LOCAL_STORAGE);
  }
  // ~~~~ SCW SDK ~~~~~

  type DoConfig = {
    tx: {
      to: string;
      data: string;
      value: number;
    };
  };

  let inputValue = "1";

  function getUsdcContract(chainId: number) {
    let contractAddress;

    switch (chainId) {
      case 97:
        contractAddress = USDC_BSC_TESTNET;
        break;
      case 421614:
        contractAddress = USDC_ARB_SEPOLIA;
        break;
      default:
        contractAddress = USDC_ARB_SEPOLIA;
    }

    return contractAddress;
  }

  function getLinkContract(chainId: number): string {
    let contractAddress;

    switch (chainId) {
      case 97:
        contractAddress = LINK_BSC_TESTNET;
        break;
      case 421614:
        contractAddress = LINK_ARB_SEPOLIA;
        break;
      default:
        contractAddress = LINK_ARB_SEPOLIA;
    }

    return contractAddress;
  }

  async function crScwSession() {
    const contractAddress = "0xFeCD581c539f8858c556Ab8FEf681975a6A25ACa";
    const functionSelector = "deposit()";
    const rules = [{
      offset: 0,
      condition: 0,
      referenceValue: "0x7a8713E21e7434dC5441Fb666D252D13F380a97d",
    }]
    const config = {
      contractAddress: getUsdcContract(scWallet.chain_id)  , //: XNYT_ARB_SEPOLIA ,//getLinkContract(scWallet.chain_id),
      functionSelector: "transfer(address,uint256)",
      validUntil: 0,
      validAfter: 0,
      valueLimit: 0,
      rules
    };

    await scWallet.createSession(config);
  }

  async function scwTx() {
    let amount = inputValue;
    //"0x84b9B910527Ad5C03A9Ca831909E21e236EA7b06"
    // XNYT_ARB_SEPOLIA; //getLinkContract(scWallet.chain_id) // 
    const erc20Address =  getUsdcContract(scWallet.chain_id) // getUsdcContract(scWallet.chain_id);
    // const toAddress = "0x7a8713E21e7434dC5441Fb666D252D13F380a97d";
    const toAddress = "0xE2Dae0f0F6EE2F3b67EC1b911D59FF92e678388e";
    const Erc20Interface = new Interface(erc20abi);

    const encodedData = Erc20Interface.encodeFunctionData("transfer", [
      toAddress,
      amount,
    ]);

    // You need to create transaction objects of the following interface
    const tx1 = {
      from: scWallet.getSCWAddress(),
      to: erc20Address, // destination smart contract address
      data: encodedData,
    };

    /// Normal txn
    // let tx = await scWallet.doTx(tx1);

    /// Session txn
    let tx = await scWallet.doTx(tx1, {
      session: true,
    });
    tx = await tx.wait();
    console.log(`Transfer done ${tx.userOpHash}`);
  }

  async function scwTxNative() {
    let amount = inputValue;
    //"0x84b9B910527Ad5C03A9Ca831909E21e236EA7b06"
    const erc20Address = XNYT_ARB_SEPOLIA; //getLinkContract(scWallet.chain_id) // getErc20Contract(scWallet.chain_id);
    const toAddress = "0xFeCD581c539f8858c556Ab8FEf681975a6A25ACa";
    const contractInterface = new Interface(sessionTestAbi);

    const encodedData = contractInterface.encodeFunctionData("deposit", []);

    // You need to create transaction objects of the following interface
    const tx1 = {
      from: scWallet.getSCWAddress(),
      to: toAddress, // destination smart contract address
      data: encodedData,
      value : amount
    };

    /// Normal txn
    // let tx = await scWallet.doTx(tx1);

    /// Session txn
    let tx = await scWallet.doTx(tx1, {
      session: "4f871131ef"// true,
    });
    tx = await tx.wait();
    console.log(`Transfer done ${tx.userOpHash}`);
  }

</script>

<main>
  <article>
    <header><h2>Wallet stats</h2></header>
    <h2>
      User Address:
      {#if userAddress}
        <span>{userAddress}</span>
      {:else}
        <span> Wallet not connected</span>
      {/if}
    </h2>
    {#if scwAddress.length > 0}
      <h3>SCW Address {scwAddress}</h3>
    {/if}
  </article>
  <article>
    <header><h5>Basic wallet methods</h5></header>
    <button on:click={connectArcana}>Connect Arcana</button>
    <button on:click={sendTx}>Send Normal Wallet</button>
  </article>

  <article>
    <header><h5>Arcana SCW method</h5></header>
    <button on:click={initGasLess}>Init Gasless Wallet</button>
    <button on:click={initSession}>Init Session</button>
    <button on:click={sendGaslessTx}>Send SCW Transaction</button>
    <button on:click={crScwSession}>Create Session through SCW SDK</button>
    <button on:click={scwTx}>✨ Txn using session</button>
    <button on:click={scwTxNative}>✨ Native token transfer using session</button>
  </article>

  <article style="width: 50%;">
    <header><h6>Values</h6></header>
    <form>
      <fieldset>
        <label>
          Amount
          <input
            name="amount"
            placeholder="Amount"
            type="text"
            bind:value={inputValue}
          />
        </label>
      </fieldset>
    </form>
  </article>
</main>
