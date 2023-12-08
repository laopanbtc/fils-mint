<template>
  <div style="width: 50%; margin: 0 auto">
    <h1>FILS-MINT</h1>
    <p>地址: {{ address }}</p>
    <p>私钥：<input style="width: 50%" type="password" v-model="privateKey"/></p>
    <p>rpc：<input style="width: 50%" type="text" v-model="web3Rpc"/></p>
    <p>gas最高价格(nanoFIL)：<input style="width: 50%" type="text" v-model="gasCap"/></p>
    <p>gas小费(nanoFIL)：<input style="width: 50%" type="text" v-model="gasTip"/></p>
    <p>张数(最好不要超过20，确认后再下一轮)：<input style="width: 50%" type="text" v-model="count"/></p>
    <p>间隔时间(秒)：<input style="width: 50%" type="text" v-model="stime"/></p>
    <h2>
      <span v-if="doing" class="deploy" @click="start">Mint中</span>
      <span v-else class="deploy" @click="start">开始</span>
    </h2>
    <h3>mint要输入私钥，请尽量用小号私钥填充，代码开源在<a href="https://github.com/minchenzz/fils-mint" target="_blank">github</a>，如担心安全问题请自行下载并部署该服务
    </h3>
    <p>author: <a href="https://twitter.com/chems_zz" target="_blank">@chems</a></p>
    <ul>
      <li v-for="tx in confirmedTxHashs">已完成交易：<a :href="'https://filfox.info/tx/'+tx" target="_blank">{{ tx }}</a>
      </li>
    </ul>

  </div>

</template>

<script setup>
import {ref} from "vue";
import {ethers, JsonRpcProvider, Wallet, formatEther, parseUnits} from "ethers";

const address = ref('')
const privateKey = ref('')
const web3Rpc = ref('https://rpc.ankr.com/filecoin')
const gasCap = ref('0.001')
const gasTip = ref('0.0001')
const count = ref(1)
const doing = ref(false)
const confirmedTxHashs = ref([])
const stime = ref(1)

const data = '0x646174613a2c7b2270223a2266696c2d3230222c226f70223a226d696e74222c227469636b223a2266696c73222c22616d74223a2231303030227d'

let providerInstance

let currentNonce

const sleep = (ms) => {
  return new Promise((resolve) => setTimeout(resolve, ms));
};

const doTx = async () => {
  let tx = {
    from: address.value,
    to: address.value,
    data: data,
    maxFeePerGas: parseUnits(gasCap.value, 'gwei'),
    maxPriorityFeePerGas: parseUnits(gasTip.value, 'gwei'),
    gasLimit: 10000000,
    value: 0,
    nonce: currentNonce,
    chainId: 314
  }
  const wallet = new Wallet(privateKey.value, providerInstance);
  const txResp = await wallet.sendTransaction(tx);
  console.log('txhash', txResp.hash)
  console.log(tx)
  confirmedTxHashs.value.push(txResp.hash.toString())
  // try {
  //   const receipt = await providerInstance.waitForTransaction(txResp.hash);
  //   console.log('tx receipt', receipt)
  //   confirmedTxHashs.value.push(txResp.hash.toString())
  // } catch (e) {
  //   console.log(e)
  // }
  currentNonce++
}

const start2 = async () => {
  providerInstance = new JsonRpcProvider(web3Rpc.value);
  const wallet = new ethers.Wallet(privateKey.value, providerInstance);
  address.value = wallet.address
  currentNonce = await providerInstance.getTransactionCount(address.value);
  if (count.value > 500) {
    count.value = 500
  }

  for (let i = 0; i < count.value; i++) {
    await doTx()
    await sleep(1000 * (stime.value + 1))
  }
}


const start = async () => {
  if (doing.value) {
    return
  }
  doing.value = true

  try {
    await start2()
  } catch (e) {
    console.log(e)
  }
  doing.value = false
}


</script>

<style scoped>

.deploy {
  cursor: pointer;
}


</style>
