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
            </div>
        </h3>
    </div>
</template>

<script>
    import {Contract, Web3Provider, Provider, utils} from "zksync-web3";
    import {ethers} from "ethers";
    // eslint-disable-next-line
    const CONTRACT_ADDRESS = "0xeFb6ca459B682056Be08Eab13B2D8bc78aE48c49"; // TODO: insert the Greeter contract address here
    // eslint-disable-next-line
    import CONTRACT_ABI from "../abi.json";

    export default {
        data() {
            return {
                chainIDs: "",
                account: "",
                toAddress: "",
                tokenId: "",
                amount: "",
                tokenData: ""

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
                    let network = await provider.getNetwork()
                    this.chainIDs = network.chainId;
                    this.signer = await provider.getSigner();
                    this.accounts = await provider.send("eth_requestAccounts", []);
                    console.log("accounts:" + this.accounts);
                    this.account = this.accounts[0];
                }
                this.initializeProviderAndSigner();
            },
            async mint() {
              let  result = await this.contract.mint(this.toAddress,this.tokenId,this.amount,this.tokenData);
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
