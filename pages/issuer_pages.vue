<template>
  <div class="top">
    <div>
      <nav id="menu">
        <ul>
          <li><a href="/" class="active-link">UserPage</a></li>
        </ul>
      </nav>
      <h1 class="title">
        Image Guardian
      </h1>
      <h2 class="title2">
        Issuer
      </h2>
      <div class="submit">
        <form v-on:submit="onSubmit">
          <div class="form"> 証明書名:
            <input type="text" placeholder="証明書名" v-model="nft.toName">
          </div>
          <div class="form">発行枚数:
            <input type="number" v-model.number="nft.issueNumber">
          </div>
          <div class="form">
            <ul>
              <li v-for="pasform in pasforms" v-bind:toAddresses="pasforms.toAddresses" v-bind:key="pasform.toAddresses">
                発行先アドレス:<input class="password" type="password" placeholder="発行先アドレス" v-model="pasforms.toAddresses">
              </li>
            </ul>
          </div>
          <input type="file" @change="captureFile"/>
          <input class="btn" type="submit"/>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import Certificate1155Contract from "~/build/contracts/ERC1155Certificate.json";
import ipfs from "~/plugins/ipfs.js";
import web3 from "~/plugins/web3.js";

export default {
  data() { 
    return { 
      write: 0,
      pasforms: [{
        id: 1,
        toAddresses: '', // 送り先アドレス
      },],
      nextPasform: 1,
      addforms: [],
      buffer: '',
      nft: {
        toName: '', // 送り先の名前
        issueNumber: 0, // 発行数
      },
    }
  }, 

methods: {
  async captureFile(event) {
    event.preventDefault()      // preventDefault()はもしイベントがキャンセル可能だったら自動でキャンセルする
    const file = await event.target.files[0]; // event.target.files でサイズ、形式などのファイル情報を取得できる
    const reader = await new window.FileReader(); // FileReader はデータ読み込みできる
    reader.readAsArrayBuffer(file);  // readAsArrayBuffer() でfileオブジェクトを読み込む
    reader.onloadend = () => { // onloadend はデータ読み込み時に発生するイベントハンドラ
      this.buffer = Buffer(reader.result);
      return event.target.result;
  }
},
  async onSubmit(event) {
    event.preventDefault()
    ipfs.files.add(this.buffer, async (error, result) => { // IPFSを用いて証明証を発行する部分
      if(error) {
        console.error(error)
        return
      }
    const accounts = await this.$web3.eth.getAccounts() // MetaMaskで使っているアカウントの取得 
    let _toAddresses = this.pasforms.toAddresses 
      // 証明証をnft化する
    let upNft = await this.$contract.methods.issueCertificate(this.nft.toName, this.nft.issueNumber, _toAddresses, result[0].hash).send({ from: accounts[0] })
      // issueCertificate関数の引数は証明証名、発行数、発行先アドレス、ipfsHashデータ
  })
},
},
}
</script>

<style>
.top {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}
#menu {
  position: absolute;
  width: 800px;
  background-color: #565656;
  font-size: 16px;
  font-family: Tahoma, Geneva, sans-serif;
  font-weight: bold;
  align-items: center;
  justify-content: center;
  text-align: center;
  top: 30px;
  left: 440px;
  padding: 8px;
  border-radius: 8px;
  -webkit-border-radius: 8px;
  -moz-border-radius: 8px;
  -o-border-radius: 8px;
}
#menu li {
  display: inline-block;
  padding: 1px;
  margin-right: 500px;
}
#menu a {
  color: #FFF;
  padding: 10px;
  text-decoration: none;
}
#menu a:hover {
  background-color: #1B191B;
  color: #FFF;
  border-radius: 20px;
  -webkit-border-radius: 20px;
  -moz-border-radius: 20px;
  -o-border-radius: 20px;
}
#menu li .active {
  background-color: #1B191B;
  color: #FFF;
  border-radius: 20px;
  -webkit-border-radius: 20px;
  -moz-border-radius: 20px;
  -o-border-radius: 20px;
}
.title {
  position: relative;
  font-family:sans-serif;
  top: 100px;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
}
.title2 {
  position: relative;
  font-family:sans-serif;
  top: 100px;
  margin-bottom: 100px;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
}
.submit {
  justify-content: center;
}
.form {
  width: 800px;
  padding: 15px;
  background:#555454;
  margin-bottom:25px;
  border-radius: 10px;
}
.form:hover{
  border:0.1px solid #04f76d;
}
.password {
  margin-left: 10px;
  margin-right: 85px;
}
.btn {
  text-align: center;
  color: rgb(3, 0, 0);
  border-radius: 10px;
}
.btn:hover {
  background-color: aquamarine;
}

</style>