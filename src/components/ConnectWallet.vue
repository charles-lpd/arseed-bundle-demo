<template>
  <div>
    <h1>{{signerAddr}}</h1>
    <button class="btn btn-danger" @click="connectMetamask" v-show="!connected">connectMetamask</button><br/>
    <br/>
    <button class="btn btn-danger" @click="connectArconnect" v-show="!connected">connectArconnect</button>
    <br/>
    <br/>
    <button class="btn btn-danger" @click="loginEverId" v-show="!connected">loginEverId</button>
    <br/>
    <br/>
    <button class="btn btn-danger" @click="test" v-show="!connected">test</button>
    <br/>
    <br/>
    <button class="btn btn-danger" @click="test2" v-show="!connected">test2</button>
    <br/>
    <br/>
    <button class="btn btn-danger" @click="test3" v-show="!connected">测试接口</button>
  </div>
</template>


<script>
// import { genArweaveAPI, genAPI } from "arseeding-js";
import pubsub from 'pubsub-js'
import Everpay from 'everpay'
import { signMessageAsync } from 'everpay/esm/lib/sign'
import { genEverId } from 'everpay/esm/utils/util'
// import { getWebAuthAPI, genAPI, genArweaveAPI } from 'arseeding-js-fido'
import { getWebAuthAPI, genAPI, genArweaveAPI } from 'arseeding-js-fido'
import base64Url from 'base64url';
import { InjectedWebauthSigner } from 'arbundles-fido/src/signing'
import { createData } from 'arbundles-fido'
import axios from 'axios'
export default {
  name: "ConnectWallet",
  data() {
    return {
      connected: false,
      signerAddr:"",
    }
  },
  methods: {
    async connectMetamask() {
      await window.ethereum.enable()
      const instance = await genAPI(window.ethereum)
      this.signerAddr = window.ethereum.selectedAddress
      this.connected = true
      instance.addr = this.signerAddr
      pubsub.publish('connected',instance)
    },
    async connectArconnect() {
      const instance = await genArweaveAPI(window.arweaveWallet)
      this.signerAddr = await window.arweaveWallet.getActiveAddress()
      this.connected = true
      instance.addr = this.signerAddr
      pubsub.publish('connected', instance)
    },
    async loginEverId(){
      try{
        const everpay = new Everpay({
          debug:true
        })
        const params = {
          everpay: everpay,
          publicKey: 'eyJJRCI6IjBIMGZjWWwwRHJyZjQveGo0L0RMYlpFajhud1JsZUMxcmFVSTN0Qm8rblU9IiwiUHVibGljS2V5IjoicFFFQ0F5WWdBU0ZZSUZYK1VaWENqRlZ2ajVucE0xME56ZzdjYmQ2bkVISjFLOVg4M3RnejFmM3VJbGdnUWFrWHJWeXptMkc0WndzUEZUSjR2dEVDcFpqTEpDelAzNlZBbWpFYVFtST0iLCJBdHRlc3RhdGlvblR5cGUiOiJwYWNrZWQiLCJUcmFuc3BvcnQiOlsiaW50ZXJuYWwiXSwiRmxhZ3MiOnsiVXNlclByZXNlbnQiOnRydWUsIlVzZXJWZXJpZmllZCI6dHJ1ZSwiQmFja3VwRWxpZ2libGUiOmZhbHNlLCJCYWNrdXBTdGF0ZSI6ZmFsc2V9LCJBdXRoZW50aWNhdG9yIjp7IkFBR1VJRCI6InJjNEFBalc4eGdwa2l3c2w4ZkJWQXc9PSIsIlNpZ25Db3VudCI6MCwiQ2xvbmVXYXJuaW5nIjpmYWxzZSwiQXR0YWNobWVudCI6InBsYXRmb3JtIn19',
          debug:true
        }
        const instance = await getWebAuthAPI(params)
        console.log(instance, 'instance')
        this.signerAddr = instance.signer.signer.account
        this.connected = true
        pubsub.publish('connected', {...instance,addr:this.signerAddr})
        // const everpayAsync = {
        //   addr:this.signerAddr,
        //   ...everpaytest
        // }

        // console.log(instance)

      }catch(e){
        console.log(e)
      }
    },
    async test(){
    const a = 'eyJJRCI6IjBIMGZjWWwwRHJyZjQveGo0L0RMYlpFajhud1JsZUMxcmFVSTN0Qm8rblU9IiwiUHVibGljS2V5IjoicFFFQ0F5WWdBU0ZZSUZYK1VaWENqRlZ2ajVucE0xME56ZzdjYmQ2bkVISjFLOVg4M3RnejFmM3VJbGdnUWFrWHJWeXptMkc0WndzUEZUSjR2dEVDcFpqTEpDelAzNlZBbWpFYVFtST0iLCJBdHRlc3RhdGlvblR5cGUiOiJwYWNrZWQiLCJUcmFuc3BvcnQiOlsiaW50ZXJuYWwiXSwiRmxhZ3MiOnsiVXNlclByZXNlbnQiOnRydWUsIlVzZXJWZXJpZmllZCI6dHJ1ZSwiQmFja3VwRWxpZ2libGUiOmZhbHNlLCJCYWNrdXBTdGF0ZSI6ZmFsc2V9LCJBdXRoZW50aWNhdG9yIjp7IkFBR1VJRCI6InJjNEFBalc4eGdwa2l3c2w4ZkJWQXc9PSIsIlNpZ25Db3VudCI6MCwiQ2xvbmVXYXJuaW5nIjpmYWxzZSwiQXR0YWNobWVudCI6InBsYXRmb3JtIn19'
    const b = genEverId('17326091635@163.com')
    // Decode the base64 URL string
    const decodedString = base64Url.decode([a,b].join(','));

    // Check the length of the decoded string
    const decodedByteLength = (decodedString).byteLength;

    // Create a new buffer with a length of 1024 bytes
    const paddedBuffer = Buffer.alloc(2048);

    // Copy the decoded string to the padded buffer
    Buffer.from(decodedString).copy(paddedBuffer);

    // If the decoded byte length is less than 1024, fill the remaining space with zeroes
    if (decodedByteLength < 2048) {
      console.log(12312312)
      paddedBuffer.fill(0, decodedByteLength);
    }

    // Log the byte length of the padded buffer
    console.log(paddedBuffer.byteLength);
    console.log(paddedBuffer)
    console.log(base64Url.toBase64(paddedBuffer))
    },
    async test2(){
      // const everpay = await new Everpay({debug:true}).smartAccountAuth('https://app-dev.permaswap.network/permalogo.svg')
      // console.log(everpay)
      const sig = await signMessageAsync({debug:true,isSmartAccount:true,account:'17326091635@163.com'},'[fujfjgjgb]12312312312312323123123,fujfjgjgbfujfjgjgbfujfjgjgbfujfjgjgbfujfjgjgbfujfjgjgb')
      console.log(sig)
      console.log(base64Url.toBuffer(sig.sig))
      console.log(Buffer.from(sig.sig))
      console.log(new Uint8Array(Buffer.from(sig.sig)))
      console.log(base64Url.toBase64(new Uint8Array(Buffer.from(sig.sig))))
      console.log(base64Url.toBase64(base64Url.toBuffer(sig.sig)))
      console.log(base64Url.toBuffer(sig), 'b')
    },
    async test3(){
      try{
        const everpay = new Everpay({
          debug:true
        })
        const params = {
          everpay: everpay,
          publicKey: 'eyJJRCI6IjBIMGZjWWwwRHJyZjQveGo0L0RMYlpFajhud1JsZUMxcmFVSTN0Qm8rblU9IiwiUHVibGljS2V5IjoicFFFQ0F5WWdBU0ZZSUZYK1VaWENqRlZ2ajVucE0xME56ZzdjYmQ2bkVISjFLOVg4M3RnejFmM3VJbGdnUWFrWHJWeXptMkc0WndzUEZUSjR2dEVDcFpqTEpDelAzNlZBbWpFYVFtST0iLCJBdHRlc3RhdGlvblR5cGUiOiJwYWNrZWQiLCJUcmFuc3BvcnQiOlsiaW50ZXJuYWwiXSwiRmxhZ3MiOnsiVXNlclByZXNlbnQiOnRydWUsIlVzZXJWZXJpZmllZCI6dHJ1ZSwiQmFja3VwRWxpZ2libGUiOmZhbHNlLCJCYWNrdXBTdGF0ZSI6ZmFsc2V9LCJBdXRoZW50aWNhdG9yIjp7IkFBR1VJRCI6InJjNEFBalc4eGdwa2l3c2w4ZkJWQXc9PSIsIlNpZ25Db3VudCI6MCwiQ2xvbmVXYXJuaW5nIjpmYWxzZSwiQXR0YWNobWVudCI6InBsYXRmb3JtIn19',
          debug:true
        }
        const instance = await this.getWebAuthAPITest(params)
        console.log(instance, 'instance')
        this.signerAddr = instance.signer.signer.account
        this.connected = true
        pubsub.publish('connected', {...instance,addr:this.signerAddr})
        // const everpayAsync = {
        //   addr:this.signerAddr,
        //   ...everpaytest
        // }

        // console.log(instance)

      }catch(e){
        console.log(e)
      }
    },
    async getWebAuthAPITest(params){
      const account = await params.everpay.smartAccountAuth('https://app-dev.permaswap.network/permalogo.svg')
      const everpayT = new Everpay({
        account,
        isSmartAccount: true
      })
     const a = {
       everpay: { ...everpayT, signMessageAsync },
       publicKey: params.publicKey,
       account: account,
       debug: params.debug
     }
     const signer = new InjectedWebauthSigner(a)
     await signer.setPublicKey()
     return {
        signer,
        async sendAndPay (arseedingUrl, data, tag, opts, needSeq, debug) {
          const dataItem = createData(
            data,
            signer,
            opts
          )
          await dataItem.sign(signer)
          const api = axios.create({ baseURL: arseedingUrl })
          const header = {
            'Content-Type': 'application/octet-stream'
          }
          if (needSeq !== undefined && needSeq) {
            header.Sort = 'true'
          }
          console.log(api, 'api')
          const tokenSymbol = tag.split('-')[1]
          console.log(tokenSymbol, 'tokenSymbol')
          console.log(dataItem.getRaw(), 'dataItem.getRaw()')
          console.log(data, 'buffer data')
          const res = await api.post(`https://seed-dev.everpay.io/bundle/tx/${tokenSymbol}`, dataItem.getRaw(), {
            headers: header,
            maxBodyLength: Infinity
          })
          console.log(res, 'res')
          console.log(debug, 'debug')
          const order = { ...res.data, tag }
          const { fee } = order
          console.log(order, 'order')
          if (+fee > 0) {
            console.log(fee, 'fee')
          }
        }
      }
    }
  }
}
</script>

<style scoped>

</style>
