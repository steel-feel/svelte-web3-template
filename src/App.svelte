<script lang="ts">
  import {
    parseEther,
    BrowserProvider,
    Interface,
    Contract,
    ethers,
  } from "ethers";
  import { onMount } from "svelte";
  
  import { SCW } from "@arcana/scw"; //From npm

  import { AuthProvider } from "@arcana/auth"; //From npm

  import { custom, createPublicClient, type EIP1193Provider } from "viem";

  import {
    createSmartAccountClient,
    BiconomySmartAccountV2,
    DEFAULT_SESSION_KEY_MANAGER_MODULE,
    createSession,
    getRandomSigner,
    createSessionSmartAccountClient,
    getSingleSessionTxParams,
  } from "@biconomy/account";

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

  async function connectViem() {
    const arcanaProvider = (await auth.connect()) as EIP1193Provider;
    const publicClient = createPublicClient({
      transport: custom(arcanaProvider),
    });
    const publicClientEth = createPublicClient({
      transport: custom(window.ethereum),
    });

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
    await scWallet.init(arcana_app_id, window.arcana.provider);
    scwAddress = await scWallet.getSCWAddress();
    //@ts-ignore
    window.scwi = scWallet;
    console.log("Address: " + scWallet.getSCWAddress());
  }

  async function sendGaslessTx() {
    let amount = inputValue;

    // const erc20Address = "0x06A0F0fa38AE42b7B3C8698e987862AfA58e90D9";
    const erc20Address = USDC_BSC_TESTNET;
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

  async function buildUserOP() {
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
      storageType: StorageType.LOCAL,
    });
  }
  const USDC_ARB_SEPOLIA = "0x75faf114eafb1BDbe2F0316DF893fd58CE46AA4d";
  const USDC_BSC_TESTNET = "0x64544969ed7EBf5f083679233325356EbE738930";

  async function crSession() {
    /// SDK Tip : get a Signer from sessionStorage, if not create a signer
    const sData = await getRandomSigner();
    // const ArbitrumSepoliaRpc = "https://rpc.ankr.com/arbitrum_sepolia"
    const BnbtestnetRpc = "https://bsc-testnet.public.blastapi.io";

    await scWallet.session.addSigner(null, {
      id: 97,
      rpcUrls: {
        97: BnbtestnetRpc,
        default: {
          http: [BnbtestnetRpc],
        },
      },
    });

    const policy = [
      {
        /** The address of the sessionKey upon which the policy is to be imparted */
        sessionKeyAddress: sData.pbKey,
        /** The address of the contract to be included in the policy */
        contractAddress: USDC_ARB_SEPOLIA,
        /** The specific function selector from the contract to be included in the policy */
        functionSelector: "transfer(address,uint256)",
        /** The list of rules which make up the policy */
        rules: [],
        /** The time interval within which the session is valid. Setting both to 0 will keep a session alive indefinitely */
        interval: {
          validUntil: 0,
          validAfter: 0,
        },
        /** The maximum value that can be transferred in a single transaction */
        valueLimit: 0n,
      },
    ];

    //@ts-ignore
    const { wait, session } = await createSession(
      scWallet.smart_account,
      policy,
      scWallet.session,
    );

    const {
      receipt: { transactionHash },
      success,
    } = await wait();

    console.log(
      `session ID ${JSON.stringify(session)} txHash ${transactionHash} success ${success}`,
    );

    await scWallet.session.updateSessionStatus(
      {
        sessionID: session.sessionIDInfo[0],
      },
      "ACTIVE",
    );
  }
  async function doSessionTx() {
    const emulatedUsersSmartAccount = await createSessionSmartAccountClient(
      {
        accountAddress: scWallet.scwAddress, // Dapp can set the account address on behalf of the user
        bundlerUrl: scWallet.smart_account.bundler.getBundlerUrl(),
        chainId: scWallet.chain_id,
      },
      scWallet.session, // Storage client, full Session or simply the smartAccount address if using default storage for your environment
    );

    const amount = parseEther("100");

    const Erc20Interface = new Interface(erc20abi);
    const toAddress = "0x7a8713E21e7434dC5441Fb666D252D13F380a97d";
    const encodedData = Erc20Interface.encodeFunctionData("transfer", [
      toAddress,
      amount,
    ]);

    const sendErc20Tx = {
      to: USDC_BSC_TESTNET,
      data: encodedData,
    };
    const BnbtestnetRpc = "https://bsc-testnet.public.blastapi.io";

    const params = await getSingleSessionTxParams(
      {
        sessionIDInfo: ["b5d0409687"],
        sessionStorageClient: scWallet.session,
      },
      {
        id: 97,
        rpcUrls: {
          97: BnbtestnetRpc,
          default: {
            http: [BnbtestnetRpc],
          },
        },
      },
      0, // index of the relevant policy leaf to the tx
    );

    const { wait } = await emulatedUsersSmartAccount.sendTransaction(
      sendErc20Tx,
      {
        ...params,
      },
    );

    const receipt = await wait();

    console.log(` 
    userOpHash : ${receipt.userOpHash} 
    txhash : ${receipt.receipt.transactionHash}`);
  }

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
  <button on:click={connectViem}>Connect Viem Wallet</button>

  <br />
  <h4>Biconomy</h4>
  <button on:click={initGasLess}>Init Gasless Wallet</button>
  <button on:click={crSession}>Create Session</button>
  <button on:click={doSessionTx}>Send Session Txn</button>

  <br />
  <h5>Arcana SCW method</h5>
  <button on:click={initSession}>Init Session</button>
  <button on:click={sendGaslessTx}>Send Gasless Transaction</button>
  <button on:click={crScwSession}>Create Session through SCW SDK</button>
  <button on:click={scwSessionTx}>Send Session Txn through SCW SDK</button>

  <br />

  <h5>Variables</h5>
  <div>
    <div>Value</div>
    <input type="text" bind:value={inputValue} />
  </div>

  <!-- Bico SCW -->
  <br />
  <h4>Bico SCW</h4>
  <button on:click={scwv4}>Init Bico SCW</button>
  <button on:click={buildUserOP}>Build User OP</button>
  <button on:click={approveBicoSCW}>Bico SCW Txn</button>
</main>
