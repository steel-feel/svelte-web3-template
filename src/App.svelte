<script lang="ts">
  import {
    parseEther,
    BrowserProvider,
    Interface,
    Contract,
    ethers,
  } from "ethers";
  import { onMount } from "svelte";

  import * as SCW from "@arcana/scw"; //From npm

  import { AuthProvider } from "@arcana/auth"; //From npm

  const erc20abi = [
    {
      inputs: [],
      stateMutability: "nonpayable",
      type: "constructor",
    },
    {
      anonymous: false,
      inputs: [
        {
          indexed: true,
          internalType: "address",
          name: "owner",
          type: "address",
        },
        {
          indexed: true,
          internalType: "address",
          name: "spender",
          type: "address",
        },
        {
          indexed: false,
          internalType: "uint256",
          name: "value",
          type: "uint256",
        },
      ],
      name: "Approval",
      type: "event",
    },
    {
      anonymous: false,
      inputs: [
        {
          indexed: true,
          internalType: "address",
          name: "previousOwner",
          type: "address",
        },
        {
          indexed: true,
          internalType: "address",
          name: "newOwner",
          type: "address",
        },
      ],
      name: "OwnershipTransferred",
      type: "event",
    },
    {
      anonymous: false,
      inputs: [
        {
          indexed: true,
          internalType: "address",
          name: "from",
          type: "address",
        },
        {
          indexed: true,
          internalType: "address",
          name: "to",
          type: "address",
        },
        {
          indexed: false,
          internalType: "uint256",
          name: "value",
          type: "uint256",
        },
      ],
      name: "Transfer",
      type: "event",
    },
    {
      inputs: [
        {
          internalType: "address",
          name: "owner",
          type: "address",
        },
        {
          internalType: "address",
          name: "spender",
          type: "address",
        },
      ],
      name: "allowance",
      outputs: [
        {
          internalType: "uint256",
          name: "",
          type: "uint256",
        },
      ],
      stateMutability: "view",
      type: "function",
    },
    {
      inputs: [
        {
          internalType: "address",
          name: "spender",
          type: "address",
        },
        {
          internalType: "uint256",
          name: "amount",
          type: "uint256",
        },
      ],
      name: "approve",
      outputs: [
        {
          internalType: "bool",
          name: "",
          type: "bool",
        },
      ],
      stateMutability: "nonpayable",
      type: "function",
    },
    {
      inputs: [
        {
          internalType: "address",
          name: "account",
          type: "address",
        },
      ],
      name: "balanceOf",
      outputs: [
        {
          internalType: "uint256",
          name: "",
          type: "uint256",
        },
      ],
      stateMutability: "view",
      type: "function",
    },
    {
      inputs: [],
      name: "decimals",
      outputs: [
        {
          internalType: "uint8",
          name: "",
          type: "uint8",
        },
      ],
      stateMutability: "view",
      type: "function",
    },
    {
      inputs: [
        {
          internalType: "address",
          name: "spender",
          type: "address",
        },
        {
          internalType: "uint256",
          name: "subtractedValue",
          type: "uint256",
        },
      ],
      name: "decreaseAllowance",
      outputs: [
        {
          internalType: "bool",
          name: "",
          type: "bool",
        },
      ],
      stateMutability: "nonpayable",
      type: "function",
    },
    {
      inputs: [
        {
          internalType: "address",
          name: "spender",
          type: "address",
        },
        {
          internalType: "uint256",
          name: "addedValue",
          type: "uint256",
        },
      ],
      name: "increaseAllowance",
      outputs: [
        {
          internalType: "bool",
          name: "",
          type: "bool",
        },
      ],
      stateMutability: "nonpayable",
      type: "function",
    },
    {
      inputs: [],
      name: "name",
      outputs: [
        {
          internalType: "string",
          name: "",
          type: "string",
        },
      ],
      stateMutability: "view",
      type: "function",
    },
    {
      inputs: [],
      name: "owner",
      outputs: [
        {
          internalType: "address",
          name: "",
          type: "address",
        },
      ],
      stateMutability: "view",
      type: "function",
    },
    {
      inputs: [],
      name: "renounceOwnership",
      outputs: [],
      stateMutability: "nonpayable",
      type: "function",
    },
    {
      inputs: [],
      name: "symbol",
      outputs: [
        {
          internalType: "string",
          name: "",
          type: "string",
        },
      ],
      stateMutability: "view",
      type: "function",
    },
    {
      inputs: [],
      name: "totalSupply",
      outputs: [
        {
          internalType: "uint256",
          name: "",
          type: "uint256",
        },
      ],
      stateMutability: "view",
      type: "function",
    },
    {
      inputs: [
        {
          internalType: "address",
          name: "to",
          type: "address",
        },
        {
          internalType: "uint256",
          name: "amount",
          type: "uint256",
        },
      ],
      name: "transfer",
      outputs: [
        {
          internalType: "bool",
          name: "",
          type: "bool",
        },
      ],
      stateMutability: "nonpayable",
      type: "function",
    },
    {
      inputs: [
        {
          internalType: "address",
          name: "from",
          type: "address",
        },
        {
          internalType: "address",
          name: "to",
          type: "address",
        },
        {
          internalType: "uint256",
          name: "amount",
          type: "uint256",
        },
      ],
      name: "transferFrom",
      outputs: [
        {
          internalType: "bool",
          name: "",
          type: "bool",
        },
      ],
      stateMutability: "nonpayable",
      type: "function",
    },
    {
      inputs: [
        {
          internalType: "address",
          name: "newOwner",
          type: "address",
        },
      ],
      name: "transferOwnership",
      outputs: [],
      stateMutability: "nonpayable",
      type: "function",
    },
  ];

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
  let scWallet;
  // let arcana_app_id = "xar_live_2f1b0f49b5682f9cab5512bf51d022f25723518a";
  // let arcana_app_id = "xar_live_e553c5570f9c4768a2656da70ecc6fd4747e7214";
  // let arcana_app_id = "xar_test_7c27043e6263eff62c6b3a348d613f5b6c9f2527";
  // let arcana_app_id = "xar_live_10df430d374e1e9505615958f9965b7fbeb894d7";
  let arcana_app_id = "xar_dev_1e3ee6a5cecc593d0dac2e1893dbe7534a174ac4"; // Arbitrum sepolia bico paymaster
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
    await scWallet.init(arcana_app_id, window.arcana.provider, undefined);
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

  type CreateSessionParam = {
    contractAddress: string;
    functionSelector: string;
    validUntil?: number;
    validAfter?: number;
    valueLimit?: number;
  };
  // ~~~~~ Session management ~~~~~~
  let sess;
  async function initSession() {
    sess = scWallet.initSession({
      storageType: 0,
    });
  }
  const USDC_ARB_SEPOLIA = "0x75faf114eafb1BDbe2F0316DF893fd58CE46AA4d";
  const USDC_BSC_TESTNET = "0x64544969ed7EBf5f083679233325356EbE738930";

  // ~~~~ SCW SDK ~~~~~

  async function crScwSession() {
    let contractAddress;

    switch (scWallet.chain_id) {
      case 97:
        contractAddress = USDC_BSC_TESTNET;
        break;
      case 421614:
        contractAddress = USDC_ARB_SEPOLIA;
        break;
      default:
        contractAddress = USDC_ARB_SEPOLIA;
    }

    const config: CreateSessionParam = {
      contractAddress,
      functionSelector: "transfer(address,uint256)",
      validUntil: 0,
      validAfter: 0,
      valueLimit: 0,
    };

    await scWallet.createSession(config);
  }

  type DoConfig = {
    tx: {
      to: string;
      data: string;
      value: number;
    };
  };

  let inputValue = "1";
  async function scwSessionTx() {
    let contractAddress;

    switch (scWallet.chain_id) {
      case 97:
        contractAddress = USDC_BSC_TESTNET;
        break;
      case 421614:
        contractAddress = USDC_ARB_SEPOLIA;
        break;
      default:
        contractAddress = USDC_ARB_SEPOLIA;
    }
    // const amount = parseEther( "0.01");
    const amount = inputValue;
    const Erc20Interface = new Interface(erc20abi);
    const toAddress = "0x7a8713E21e7434dC5441Fb666D252D13F380a97d";
    const encodedData = Erc20Interface.encodeFunctionData("transfer", [
      toAddress,
      amount,
    ]);

    const sendErc20Tx = {
      to: contractAddress,
      data: encodedData,
      value: 0,
    };

    await scWallet.doSessionTx(sendErc20Tx);
  }

  function getErc20Contract(chainId: number) {
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

  async function scwTx2() {
    let amount = inputValue;

    const erc20Address = getErc20Contract(scWallet.chain_id);
    const toAddress = "0x7a8713E21e7434dC5441Fb666D252D13F380a97d";
    const Erc20Interface = new Interface(erc20abi);

    const encodedData = Erc20Interface.encodeFunctionData("transfer", [
      toAddress,
      BigInt(amount),
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
    let tx = await scWallet.doTx(tx1);
    tx = await tx.wait();
    console.log(`Transfer done ${tx.userOpHash}`);
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
  {#if scwAddress.length > 0}
    <h3>SCW Address {scwAddress}</h3>
  {/if}
  <button on:click={connectArcana}>Connect Arcana</button>
  <button on:click={sendTx}>Send Normal Wallet</button>

  <br />
  <h5>Arcana SCW method</h5>
  <button on:click={initGasLess}>Init Gasless Wallet</button>
  <button on:click={initSession}>Init Session</button>
  <button on:click={sendGaslessTx}>Send Gasless Transaction</button>
  <button on:click={crScwSession}>Create Session through SCW SDK</button>
  <button on:click={scwSessionTx}>Send Session Txn through SCW SDK</button>

  <button on:click={scwTx2}>✨ doTx2</button>

  <br />

  <h5>Variables</h5>
  <div>
    <div>Value</div>
    <input type="text" bind:value={inputValue} />
  </div>
</main>
