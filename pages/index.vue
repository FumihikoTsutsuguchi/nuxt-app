<template>
  <section class="container">
    <div class="login" @click="doLogin">
      <button @click="doLogin">ログイン</button>
      [login:{{ user_name }}]
    </div>
    <div>
      <button @click="logout">ログアウト</button>
    </div>
    <h1>{{ title }}</h1>
    <p class="message">{{ message }}</p>
    <div v-if="logined"></div>
    <table>
      <tr>
        <th>Message</th>
        <td><input type="text" v-model="msg" size="50"></td>
        <td><button @click="add">投稿</button></td>
      </tr>
    </table>
    <ul v-for="(data, key) in json_data">
      <li>
        <div class="list1">{{ data.msg }}</div>
        <div class="list2">{{ data.user }}{{ data.posted }}</div>
      </li>
    </ul>
  </section>
</template>

<script>
import firebase from 'firebase/compat/app'
import 'firebase/compat/auth'
import 'firebase/compat/database'

// const axios = require('axios');

const firebaseConfig = {
  apiKey: "AIzaSyBIG3pNB45joPF2UPeRPwO0lcSowBRIpOA",
  authDomain: "nuxt-app-demo-69ddc.firebaseapp.com",
  projectId: "nuxt-app-demo-69ddc",
  storageBucket: "nuxt-app-demo-69ddc.firebasestorage.app",
  messagingSenderId: "820669731922",
  appId: "1:820669731922:web:0a6e09922375df4443bb8b",
  measurementId: "G-98J8L03MQW"
};

if(!firebase.apps.length)firebase.initializeApp(firebaseConfig)

export default {
  data() {
    return {
      title:'Board',
      message:'ミニ伝言板、最新の投稿を表示します。',
      user_name:'',
      logined:true,
      msg:'',
      user: null,
      page:0,
      num_per_page:10,
      json_data:{}
    };
  },
  methods:{
    login() {
      const provider = new firebase.auth.GoogleAuthProvider();
      firebase.auth().signInWithPopup(provider).then(result => {
        console.log(result.user);
        this.user = result.user;
        this.user_name = result.user.displayName;
        this.message = 'ログインしました。';
        let db = firebase.database();
        let ref = db.ref('board');
        ref.orderByKey().limitToLast(this.num_per_page).on('value', (snapshot => {
          if (firebase.auth().currentUser != null) {
            let arr =[];
            let data = snapshot.val();
            for(let item in data) {
              arr.unshift(data[item]);
            }
            console.log(arr);
            this.json_data =  arr;
          }else {
            this.json_data = {};
          }
        }))
      }).catch(err => { this.message = err.message });
    },
    logout() {
      firebase.auth().signOut();
      this.user_name = '';
      this.json_data =  {};
      this.message = 'ログアウトしました。';
    },
    doLogin() {
      if (firebase.auth().currentUser == null) {
        this.login();
      }else {
        this.logout();
      }
    },
    add() {
      if (firebase.auth().currentUser == null) {
        alert('ログインしないと投稿できません。');
        return;
      }
      let db = firebase.database();
      let user = firebase.auth().currentUser;
      console.log(user);
      let ref = db.ref('board');
      let d = new Date();
      let dstr = d.getFullYear() + '-' + (d.getMonth() + 1) + '-' + d.getDate() + ' ' + d.getHours() + ':' + d.getMinutes() + ':' + d.getSeconds();
      let id = d.getTime();
      let data = {
        msg:this.msg,
        user:user.displayName,
        posted:dstr,
      };
      firebase.database().ref('board/' + id).set(data);
      this.msg = '';
      this.message = '投稿しました。';
    },
  },
  created() {
    if (firebase.auth().currentUser == null) {
      this.login();
    }
    console.log(firebase.auth().currentUser);
  }
}

</script>

<style>
.login {
  font-weight: bold;
  font-size: 12px;
  cursor: pointer;
}
.list1 {
  text-align: left;
  font-size: 16px;
}
.list2 {
  text-align: right;
  font-size: 10px;
}
.container {
  padding: 5px 10px;
}
h1 {
  font-size: 60px;
  color: #345980;
}
p {
  padding-top: 5px;
  margin: 10px 0px;
  font-size: 20px;
}
.message {
  font-size: 16px;
}
div {
  font-size: 14px;
}
ul {
  margin: 0 10px;
}
li {
  padding: 10px;
  font-size: 14px;
}
tr th {
  width: 120px;
  background-color: darkblue;
  color: white;
  font-size: 14px;
}
tr td {
  padding: 5px 10px;
  background-color: #eef;
  font-size: 14px;
}
input {
  font-size: 14px;
}
button {
  font-size: 14px;
}
</style>