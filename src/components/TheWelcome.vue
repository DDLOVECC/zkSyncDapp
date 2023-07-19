<template>
  <div class="greetings">
    <h3>
      <div>
        <button @click="initWallet">Connect wallet</button>
      </div>
      <div>
        ChainID: {{chainIDs}}<br/>
        Current Account: {{account}}<br/>
      </div>
      <div>
        address：<input v-model="toAddress"> <br/>
        tokenId：<input v-model="tokenId"> <br/>
        amount：<input v-model="amount"><br/>
        tokenData：<input v-model="tokenData"><br/>
        <button @click="mint">mint</button>
        <button @click="test">test</button>
      </div>
    </h3>
  </div>
</template>

<script>
  import {Contract, Web3Provider, Provider, utils} from "zksync-web3";
  import {ethers} from "ethers";
  // eslint-disable-next-line
  const CONTRACT_ADDRESS = "0xB71fF6775eAb76279f6B27e5cACe8e16a2B1ECEa"; // TODO: insert the Greeter contract address here
 // const CONTRACT_ADDRESS = "0xe3ABdb725393D672C7149e238a6Fe4C3E92b9C0e";
  // eslint-disable-next-line
  import CONTRACT_ABI from "../AiYue.json";

  export default {
    data() {
      return {
        chainIDs: "",
        account: "",
        toAddress: "",
        tokenId: "",
        amount: "",
        tokenData: "",
        nonce:""

      }
    },
    methods: {
      async initWallet() {
        let provider;
        if (window.ethereum == null) {
          console.log("MetaMask not installed; using read-only defaults")
          provider = ethers.getDefaultProvider();
        } else {
          provider = new ethers.providers.Web3Provider(window.ethereum);
          let network = await provider.getNetwork();
          this.chainIDs = network.chainId;
          this.signer = await provider.getSigner();
          this.accounts = await provider.send("eth_requestAccounts", []);
          console.log("accounts:" + this.accounts);
          this.account = this.accounts[0];
          let nonce = await provider.getTransactionCount(this.account);
          this.nonce = nonce;
          console.log("nonce:" + nonce);
        }
        this.initializeProviderAndSigner();
      },
      async test(){
        let  result = await this.contract.forward("0x046daf2de2958316a64deae5450a01b4f3464086",25,"0xCe4A88DC0c45FFEb8b16c8b7B3Ba552addf7ebeA","0x73c968cf000000000000000000000000046daf2de2958316a64deae5450a01b4f34640860000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000800000000000000000000000000000000000000000000000000000000000000000","0x04cf74aed4819d4601a73762168670454c21ea67f1fd54b714c6ea3a9bf11c945c36e3cdcb8e9d572e65735e1c6ef1b1e2a350eb90ab92ec15908d874b801b661b");
        console.log("result:" + JSON.stringify(result));
      },
      async mint() {
        let calldata = "";

        // get the function signature by hashing it and retrieving the first 4 bytes
        //mint(address account, uint256 id, uint256 amount, bytes memory data)
        console.log("fnSignature starting");
        let fnSignature = ethers.utils.keccak256(ethers.utils.toUtf8Bytes("mint(address, uint256, uint256, bytes)")).substr(0,10);
        console.log("fnSignature：{}",fnSignature);
        // encode the function parameters and add them to the call data
        let fnParams =  ethers.utils.defaultAbiCoder.encode(
                ["address","uint256","uint256","bytes"],
                [this.toAddress,this.tokenId,this.amount,this.tokenData]
        );
        calldata = fnSignature + fnParams.substr(2);
        //address sender, uint nonce, bytes calldata _data
        let  messageHash = await this.contract.getHash(this.account,this.nonce,calldata);
        console.log("hash:{}" + messageHash);
        const signature = await this.signer.signMessage(ethers.utils.arrayify(messageHash));
        console.log("signature:{}" + signature);
        console.log("account:{};", this.account);
        console.log("nonce:{};", this.nonce);
        console.log("calldata:{};", calldata);
        let  result = await this.contract.forward(this.account,this.nonce,"0x31201F33e35BC203C8B419AF3cCbb610E8DF631d",calldata,signature);
        //address sender,uint nonce, address _to, bytes calldata _data, bytes memory _signature
        console.log("result:" + JSON.stringify(result));
      },
      initializeProviderAndSigner() {
        //this.provider = new Provider('https://testnet.era.zksync.dev');
        // Note that we still need to get the Metamask signer
        this.signer = (new Web3Provider(window.ethereum)).getSigner();
        this.contract = new Contract(
                CONTRACT_ADDRESS,
                CONTRACT_ABI,
                this.signer
        );
      },

    }
  }
</script>
<style scoped>
  h1 {
    font-weight: 500;
    font-size: 2.6rem;
    top: -10px;
  }

  h3 {
    font-size: 1.2rem;
  }

  .greetings h1,
  .greetings h3 {
    text-align: center;
  }

  @media (min-width: 1024px) {
    .greetings h1,
    .greetings h3 {
      text-align: left;
    }
  }
</style>
