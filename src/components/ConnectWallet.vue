<template>
  <div>
    <h1>{{signerAddr}}</h1>
    <button class="btn btn-danger" @click="connectMetamask" v-show="!connected">connectMetamask</button><br/>
    <br/>
    <button class="btn btn-danger" @click="connectArconnect" v-show="!connected">connectArconnect</button>
    <br/>
    <br/>
    <button class="btn btn-danger" @click="loginEverId" v-show="!connected">loginEverId</button>
  </div>
</template>


<script>
// import { genArweaveAPI, genAPI } from "arseeding-js";
import pubsub from 'pubsub-js'
import Everpay from 'everpay'
import { getWebAuthAPI, genAPI, genArweaveAPI } from 'arseeding-js-fido'
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
    }
  }
}
</script>

<style scoped>

</style>
