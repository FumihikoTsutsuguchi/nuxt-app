<template>
  <section class="container">
    <div v-if="!logined" class="login" @click="doLogin">
      <button @click="doLogin">ログイン</button>
    </div>
    <div v-if="logined">
      <button @click="logout">ログアウト</button>
      [ユーザー名: {{ user_name }}]
    </div>
    <h1>{{ title }}</h1>
    <p class="message">{{ message }}</p>
    <table>
      <tr>
        <th>Message</th>
        <td><input type="text" v-model="msg" size="50"></td>
        <td><button @click="add">投稿</button></td>
      </tr>
    </table>
    <ul>
      <li v-for="(data, i) in json_data" :key="data.id">
        <div class="list1">
          <span class="list1-no">{{ `NO${i + 1}:` }}</span>
          <span>{{ data.msg }}</span>
        </div>
        <div class="list2">{{ data.user }} {{ data.posted }}</div>
        <div class="list3">
          <button @click="deleteItem(data.id)">×</button>
        </div>
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
      title:'掲示板アプリ',
      message:'ログイン後、最新の投稿を表示します。',
      user_name:'',
      logined: false,
      msg:'',
      user: null,
      page: 0,
      num_per_page: 10,
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
        this.logined = true;
        let db = firebase.database();
        let ref = db.ref('board');
        ref.orderByKey().limitToLast(this.num_per_page).on('value', (snapshot => {
          if (firebase.auth().currentUser != null) {
            const raw = snapshot.val() || {}
            const arr = []
            for (const id in raw) {
               const obj = raw[id]
               arr.unshift({ id, ...obj })  // 時系列逆転
             }
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
      this.logined = false;
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
      let user = firebase.auth().currentUser;
      let d = new Date();
      let dstr = d.getFullYear() + '-' + (d.getMonth() + 1) + '-' + d.getDate() + ' ' + d.getHours().toString().padStart(2, '0') + ':' + d.getMinutes().toString().padStart(2, '0') + ':' + d.getSeconds().toString().padStart(2, '0');
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
    deleteItem(id) {
      if (!firebase.auth().currentUser) {
        alert('ログインしないと削除できません。')
        return
      }
      try {
        firebase.database().ref(`board/${id}`).remove()
        // 2) 画面側の配列からも除外
        this.json_data = this.json_data.filter(item => item.id !== id)
        this.message = '投稿を削除しました。'
      } catch (err) {
        console.error(err)
        this.message = '削除に失敗しました：' + err.message
      }
    }
  },
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
  margin-left: auto;
  padding-right: 30px;
}
.list3 {
  position: absolute;
  right: 5px;
  text-align: right;
  font-size: 10px;
}
.container {
  padding: 5px 10px;
  max-width: 980px;
  margin: 0 auto;
}
h1 {
  font-size: 40px;
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
  padding: 0 !important;
  margin: 0 !important;
}
li {
  position: relative;
  display: flex;
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
.list1-no {
  font-size: 12px;
}
</style>