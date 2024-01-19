<template>
  <div class="upload-img-root">
    <el-select v-show="connected" v-model="selectedSymbol" class="m-2" placeholder="Select" size="large">
      <el-option
        v-for="(item,index) in symbols"
        :key="index"
        :label="`${item.symbol} ${balanceStack[item.symbol]}`"
        :value="item.symbol"
      />
    </el-select>
    <span v-if="connected">{{balance}}</span>
    <el-upload v-show="connected"
        :file-list="fileList"
        :show-file-list="false"
        :auto-upload="false"
        :on-change="handleChangeFileSuccess"
        :on-success="handleAvatarSuccess"
        drag
        multiple
        action="#"
        class="avatar-uploader"
    >
      <i class="el-icon-upload"/>
      <div class="el-upload__text">Drag the file here，or<em style="color: #031425;"> Click to upload</em></div>
    </el-upload>
    <div>
      <ul>
        <li v-for="(order, index) in orders" :key="index">
          <a style="margin-right:10px;" target="_blank" :href="`${arseedUrl}/${order.itemId}`">{{order.itemId}}</a>
          <a target="_blank" :href="`https://arweave.net/${order.itemId}`">By arweave gateway(when onChainStatus pending or success)</a>
          <div>{{JSON.stringify(order, null, 2)}}</div>
        </li>
      </ul>
    </div>
    <button v-if="connected" @click="testSignMessageAsync">
    testSignMessageAsync
    </button>
  </div>
</template>

<script>
import pubsub from 'pubsub-js'
import Everpay from 'everpay'
import { getBundleFee, getOrders, getTokenTagByEver } from 'arseeding-js'
import Bignumber from 'bignumber.js'
import { genEverId, isSmartAccount } from 'everpay/esm/utils/util'
import { getWebAuthAPI } from 'arseeding-js-fido'
function  getArseedUrl() {
  let arseedUrl = "https://arseed.web3infra.dev"
  const hostname = window.location.hostname
  if ( hostname.split(".")[0].indexOf("dev") !== -1 || hostname === "localhost") { // test env
    arseedUrl = "https://arseed.web3infra.dev"
  }
  return arseedUrl
}

export default {
  name: 'Upload',
  data() {
    return {
      fileList: [],
      submitResp: "",
      selectedSymbol: '',
      symbols: [],
      connected: false,
      instance: {},
      everpay: {},
      balance: '',
      orders: [],
      balanceStack: {},
      arseedUrl: getArseedUrl()
    };
  },
  watch: {
    async selectedSymbol() {
      console.log(this.balance, '1', this.selectedSymbol, '2',  this.instance.addr)
      if (this.selectedSymbol && this.instance.addr) {
       const token = await getTokenTagByEver(this.selectedSymbol)
       console.log(token[0])
        this.everpay.balance({
          tag: token[0],
          account: this.instance.addr
        }).then(result => {
          console.log(result)
          this.balance = result
        })
      }
    }
  },
  methods: {
    handleChangeFileSuccess(file, fileList) {
      this.combineFileList(fileList);
    },
    handleAvatarSuccess(res, file, fileList) {
      this.combineFileList(fileList);
    },
    async combineFileList(files) {
      const file = files[files.length - 1]
      console.log('file.size', file.size)
      const token = this.symbols.find((t)=> t.symbol === this.selectedSymbol)
      const fee = await getBundleFee(this.arseedUrl, file.size, token.symbol)
      const formatedFee = new Bignumber(fee.finalFee).dividedBy(new Bignumber(10).pow(fee.decimals)).toString()
      console.log(formatedFee, 'formatedFee')
      console.log(this.balance, 'this.balance')
      if (+this.balance >= +formatedFee) {
        const reader = new FileReader();
        reader.readAsArrayBuffer(file.raw);
        reader.onload= async ()=> {
          const data = reader.result
          const ops = {
            tags: [{name: "FileName", value:file.name},{name: "Content-Type",value:file.raw.type},{name:"eid", value: genEverId(this.instance.addr)}]
          }
          const res = await this.instance.sendAndPay(this.arseedUrl, data, token.tag, ops)
          console.log(res, 'res demo')
          this.submitResp = JSON.stringify(res)
          this.getOrders()
        }
      } else {
        alert(`need ${formatedFee} ${token.symbol} to upload`)
      }
    },
    async getOrders() {
      const acc  = isSmartAccount(this.instance.addr) ? genEverId(this.instance.addr) : this.instance.addr
      getOrders(this.arseedUrl, acc).then(orders => {
        this.orders = orders
      })
    },
    intervalUpdateOrders () {
      setTimeout(() => {
        this.getOrders()
        this.intervalUpdateOrders()
      }, 10000)
    },
    async getBalances() {
      this.everpay.balances({
        account: this.instance.addr
      }).then(balances => {
        const balanceStack = {}
        balances.forEach(balanceItem => {
          balanceStack[balanceItem.symbol] = balanceItem.balance
        })
        this.balanceStack = balanceStack
      })
    },
    async testSignMessageAsync() {
      const params = {
        everpay: this.instance,
        publicKey: 'eyJJRCI6InVCVithc3h0a3JrU0xmb3pqVkNuL1E9PSIsIlB1YmxpY0tleSI6InBRRUNBeVlnQVNGWUlJeWQ0ckhxbjlQUHFaZW5wZ1Erbk95Nmp3VFozK08zUVhIbVZ2ZzJjQnpPSWxnZ3E2d21XVmxNSGFPUFNGSjBHU2ovdU00TEtUVjg1SDc4NUZPOThkM09oS0E9IiwiQXR0ZXN0YXRpb25UeXBlIjoibm9uZSIsIlRyYW5zcG9ydCI6WyJpbnRlcm5hbCJdLCJGbGFncyI6eyJVc2VyUHJlc2VudCI6dHJ1ZSwiVXNlclZlcmlmaWVkIjp0cnVlLCJCYWNrdXBFbGlnaWJsZSI6dHJ1ZSwiQmFja3VwU3RhdGUiOnRydWV9LCJBdXRoZW50aWNhdG9yIjp7IkFBR1VJRCI6InV0cFZacWVxUUIrOWxrVmhtbFVTRFE9PSIsIlNpZ25Db3VudCI6MCwiQ2xvbmVXYXJuaW5nIjpmYWxzZSwiQXR0YWNobWVudCI6InBsYXRmb3JtIn19',
        account: this.instance.addr
      }
      const data = await getWebAuthAPI(params)
      console.log(data, 'data')
      const options = {
        tags: [
          { name: 'Content-Type', value: 'text/plain' },
          { name: 'aa', value: 'aaa' }
        ]
      }
      const res = await data.sendAndPay(this.arseedUrl, 'submit data', 'arweave,ethereum-ar-AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA,0x4fadc7a98f2dc96510e42dd1a74141eeae0c1543', options)
      console.log(res, 'res')
      // const state = await this.instance.signMessageAsync({isSmartAccount:true,debug:true, account: this.instance.addr},'submit data')
      // console.log(state, 'testSignMessageAsync data')
    }
  },
  async mounted() {
    this.everpay = new Everpay({debug:true})
    console.log(this.everpay)
   const info = await  this.everpay.info()
    console.log(info, 'info')
    this.symbols = info.tokenList.map(token => {
      return {
        symbol: token.symbol,
        tag: token.tag
      }
    })
    this.selectedSymbol = this.symbols[0].symbol
    console.log(this.selectedSymbol, 'info')

    this.pubId = pubsub.subscribe('connected',async (msgName,data)=>{
      this.connected = true
      this.instance = data
      this.getOrders()
      this.intervalUpdateOrders()
      this.getBalances()
    })
  },
  beforeDestroy() {
    pubsub.unsubscribe(this.pubId)
  },
};
</script>

