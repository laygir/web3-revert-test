<template>
  <div id="app">
    <button @click="connect">Connect</button>
  </div>
</template>

<script>
import Web3 from "web3";
import CallRevert from "./CallRevert.json";

const web3 = new Web3(Web3.givenProvider || "http://localhost:8545");

export default {
  name: "App",
  methods: {
    async connect() {
      try {
        const account = web3.eth.accounts.create("my-pass-phrase");
        web3.eth.accounts.wallet.add(account);
        const accounts = await web3.eth.getAccounts();

        await web3.eth.sendTransaction({
          from: accounts[0],
          to: account.address,
          value: web3.utils.toWei("0.01", "ether"),
        });

        console.log(
          "%%%%% account balance: ",
          await web3.eth.getBalance(account.address)
        );

        const contract = new web3.eth.Contract(CallRevert.abi, {
          data: CallRevert.bytecode,
        });

        const contractInstance = await contract
          .deploy({
            arguments: [],
          })
          .send({
            from: account.address,
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
