<template>
    <div class="greetings">
        <div class="connect">
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
        <div class="approveAll">
            <div class="approve">
                <h2 class="usdt">
                    质押 USDT
                </h2>
                <h2>质押目标地址(质押授权给谁)：<input v-model="approve_address"></h2> <br/>
                <h2>质押数量：<input v-model="approve_amount"></h2><br/>
                <button @click="approve">approve</button>
            </div>
        </div>
    </div>
</template>

<script>
    import {Contract, Web3Provider, Provider, utils} from "zksync-web3";
    import {ethers} from "ethers";
    // eslint-disable-next-line
    //const CONTRACT_ADDRESS = "0xeFb6ca459B682056Be08Eab13B2D8bc78aE48c49"; // TODO: insert the Greeter contract address here
    const CONTRACT_ADDRESS = "0xD14f4B9FD4d788E027b360A4536E8Ada9dEd0d44";
    // eslint-disable-next-line
    import CONTRACT_ABI from "../new.json";

    const USDT_CONTRACT_ADDRESS = "0x0E2f0A595D7A213440e30de76E0dA4426203Cf04";
    // eslint-disable-next-line
    import USDT_CONTRACT_ABI from "../usdtAbi.json";

    export default {
        data() {
            return {
                chainIDs: "",
                account: "",
                toAddress: "",
                tokenId: "",
                amount: "",
                tokenData: "",
                contract: "",
                usdtContract: "",
                approve_address: "",
                approve_amount: "",
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
                let result = await contract.mint(this.toAddress, this.tokenId, this.amount, this.tokenData);
                console.log("result:" + JSON.stringify(result));
            },
            async approve() {
                console.log("usdtContract:" + this.usdtContract);
                let result = await this.usdtContract.approve(this.approve_address, this.approve_amount);
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
                this.usdtContract = new Contract(
                    USDT_CONTRACT_ADDRESS,
                    USDT_CONTRACT_ABI,
                    this.signer
                );
                console.log("===:{}", this.usdtContract);
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

    .greetings {
        display: flex;
        flex-direction: row;
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

    .connect {
        border-style: solid;
        padding: 20px;
    }

    .approveAll {
        border-style: solid;
        padding: 20px;
        margin-left: 20px;
    }
</style>
