<script>
  import { parseEther, BrowserProvider, Interface, Contract } from "ethers";
  import { onMount } from "svelte";
  import {SCW} from "@arcana/scw"; //From npm
  import { AuthProvider } from "@arcana/auth"; //From npm

  import { createSmartAccountClient } from "@biconomy/account";
    import { debug } from "svelte/internal";

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
  let arcana_app_id = "xar_dev_9ace66ce806cdf98ad45d6d09ef57172201e4f1f"; // Arbitrum sepolia bico paymaster
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

    console.log("Auth", auth);
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
    console.log("window.arcana.provider", window.arcana.provider);
    await scWallet.init(arcana_app_id, window.arcana.provider);
    console.log("Address: " + scWallet.getSCWAddress());
  }

  async function sendGaslessTx() {
    let amount = 0.1;

    const erc20Address = "0x06A0F0fa38AE42b7B3C8698e987862AfA58e90D9";
    const toAddress = "0x7a8713E21e7434dC5441Fb666D252D13F380a97d";
    const Erc20Interface = new Interface(erc20abi);

    const encodedData = Erc20Interface.encodeFunctionData("approve", [
      toAddress,
      parseEther(amount + ""),
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
  let biconomySmartAccount;
  let scwAddress = "";
  async function scwv4() {
    biconomySmartAccount = await createSmartAccountClient({
      signer: wallet,
      bundlerUrl:
        "https://bundler.biconomy.io/api/v2/97/nJPK7B3ru.dd7f7861-190d-41bd-af80-6877f74b8f44", // From dashboard.biconomy.io
    });

    // smartAccount.
    // debugger;
    scwAddress = await biconomySmartAccount.getAccountAddress({
      index: 0,
    });
  }

  async function buildUserOP(){
    const toAddress = "0x7a8713E21e7434dC5441Fb666D252D13F380a97d";
    //USDT Arbitrum Sepolia 0x9aA40Cc99973d8407a2AE7B2237d26E615EcaFd2
    const erc20Address = "0x9aA40Cc99973d8407a2AE7B2237d26E615EcaFd2";
    const tokenContract = new Contract(
      // polygon  usdc address
      erc20Address,
      erc20abi,
    );
    const usdcAmount = parseEther("0.1");
    const { data } = await tokenContract.approve.populateTransaction(
      toAddress,
      usdcAmount,
    );
    const tx1 = {
      to: tokenContract.address, //erc20 token address
      value: "0",
      data,
    };

    let userOp = await biconomySmartAccount.buildUserOp([tx1]);
    console.log(`asdasa ${JSON.stringify(userOp)}`);
    
    
  }

  async function approveBicoSCW() {
    const toAddress = "0x7a8713E21e7434dC5441Fb666D252D13F380a97d";
    //USDT Arbitrum Sepolia 0x9aA40Cc99973d8407a2AE7B2237d26E615EcaFd2
    const erc20Address = "0x9aA40Cc99973d8407a2AE7B2237d26E615EcaFd2";
    const tokenContract = new Contract(
      // polygon  usdc address
      erc20Address,
      erc20abi,
    );
    const usdcAmount = parseEther("0.1");
    const { data } = await tokenContract.approve.populateTransaction(
      toAddress,
      usdcAmount,
    );
    const tx1 = {
      to: tokenContract.address, //erc20 token address
      value: "0",
      data,
    };

    let userOp = await biconomySmartAccount.buildUserOp([tx1]);
    console.log(`asdasa ${JSON.stringify(userOp)}`);

    userOp.paymasterAndData =
      "0xC8d7b368D47994F4fC300C6FF2958d7E990e37D4000000000000000000000000C8d7b368D47994F4fC300C6FF2958d7E990e37D4";

    const userOpResponse = await biconomySmartAccount.sendUserOp(userOp);

    console.log("userOpHash", userOpResponse);
    const { receipt } = await userOpResponse.wait(1);
    console.log("txHash", receipt.transactionHash);
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

  <button on:click={connectArcana}>Connect Arcana</button>
  <button on:click={initGasLess}>Connect Gasless Wallet</button>
  <br />
  <button on:click={sendTx}>Send Normal Wallet</button>
  <button on:click={sendGaslessTx}>Send Gasless Transaction</button>

  <!-- Bico SCW -->
  <br />
  <h4>Bico SCW</h4>
  <button on:click={scwv4}>Init Bico SCW</button>
  <button on:click={buildUserOP}>Build User OP</button>
  <button on:click={approveBicoSCW}>Bico SCW Txn</button>
  {#if scwAddress.length > 0}
    <h3>SCW Address {scwAddress}</h3>
  {/if}
</main>
