<template>
  <div id="app">
    <button @click="connect">Connect using Alchemy</button>
  </div>
</template>

<script>
// import Web3 from "web3";
import { createAlchemyWeb3 } from "@alch/alchemy-web3";

import CallRevert from "./CallRevert.json";

// const web3 = new Web3(Web3.givenProvider || "http://localhost:8545");
const web3 = createAlchemyWeb3(process.env.VUE_APP_ALCHEMY_KEY);
web3.eth.handleRevert = true;

export default {
  name: "App",
  created() {
    if (typeof window.ethereum !== "undefined") {
      console.log("MetaMask is installed!");
    } else {
      console.log("MetaMask is NOT installed!");
    }
  },
  methods: {
    async connect() {
      try {
        const accounts = await web3.eth.getAccounts();

        console.log("accounts", accounts);

        const contract = new web3.eth.Contract(CallRevert.abi, {
          data: CallRevert.bytecode,
        });

        const contractInstance = await contract
          .deploy({
            arguments: [],
          })
          .send({
            from: accounts[0],
            gas: 2850000,
          });

        console.log(
          "%%%%% contract address: ",
          contractInstance.options.address
        );

        const tx = {
          from: accounts[0],
          gas: 2850000,
        };

        contractInstance.handleRevert = true;
        await contractInstance.methods.Start(102).send(tx);
      } catch (e) {
        console.error("%%%%% catches error: ", e);
      }
    },
  },
};
</script>
