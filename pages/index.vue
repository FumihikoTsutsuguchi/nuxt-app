<template>
  <section class="container">
    <div v-if="!logined" class="login">
      <button class="gsi-material-button" @click="doLogin">
        <div class="gsi-material-button-state"></div>
        <div class="gsi-material-button-content-wrapper">
          <div class="gsi-material-button-icon">
            <svg
              version="1.1"
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 48 48"
              xmlns:xlink="http://www.w3.org/1999/xlink"
              style="display: block"
            >
              <path
                fill="#EA4335"
                d="M24 9.5c3.54 0 6.71 1.22 9.21 3.6l6.85-6.85C35.9 2.38 30.47 0 24 0 14.62 0 6.51 5.38 2.56 13.22l7.98 6.19C12.43 13.72 17.74 9.5 24 9.5z"
              ></path>
              <path
                fill="#4285F4"
                d="M46.98 24.55c0-1.57-.15-3.09-.38-4.55H24v9.02h12.94c-.58 2.96-2.26 5.48-4.78 7.18l7.73 6c4.51-4.18 7.09-10.36 7.09-17.65z"
              ></path>
              <path
                fill="#FBBC05"
                d="M10.53 28.59c-.48-1.45-.76-2.99-.76-4.59s.27-3.14.76-4.59l-7.98-6.19C.92 16.46 0 20.12 0 24c0 3.88.92 7.54 2.56 10.78l7.97-6.19z"
              ></path>
              <path
                fill="#34A853"
                d="M24 48c6.48 0 11.93-2.13 15.89-5.81l-7.73-6c-2.15 1.45-4.92 2.3-8.16 2.3-6.26 0-11.57-4.22-13.47-9.91l-7.98 6.19C6.51 42.62 14.62 48 24 48z"
              ></path>
              <path fill="none" d="M0 0h48v48H0z"></path>
            </svg>
          </div>
          <span class="gsi-material-button-contents">Sign in with Google</span>
          <span style="display: none">Sign in with Google</span>
        </div>
      </button>
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
        <td><input type="text" v-model="msg" size="50" /></td>
        <td><button @click="add">投稿</button></td>
      </tr>
    </table>
    <div class="board">
      <ul>
        <li v-for="(data, i) in pagedData" :key="data.id">
          <div class="list1">
            <span class="list1-no">{{ `NO${currentPage * num_per_page + i + 1}:` }}</span>

            <span>{{ data.msg }}</span>
          </div>
          <div class="list2">{{ data.user }} {{ data.posted }}</div>
          <div class="list3">
            <button @click="deleteItem(data.id)">×</button>
          </div>
        </li>
      </ul>
      <nav v-if="totalPages > 1" class="pager">
        <button @click="prevPage" :disabled="currentPage === 0">Prev</button>
        <span>{{ currentPage + 1 }} / {{ totalPages }}</span>
        <button @click="nextPage" :disabled="currentPage >= totalPages - 1">
          Next
        </button>
      </nav>
    </div>
  </section>
</template>

<script>
import firebase from "firebase/compat/app";
import "firebase/compat/auth";
import "firebase/compat/database";

// const axios = require('axios');

const firebaseConfig = {
  apiKey: "AIzaSyBIG3pNB45joPF2UPeRPwO0lcSowBRIpOA",
  authDomain: "nuxt-app-demo-69ddc.firebaseapp.com",
  projectId: "nuxt-app-demo-69ddc",
  storageBucket: "nuxt-app-demo-69ddc.firebasestorage.app",
  messagingSenderId: "820669731922",
  appId: "1:820669731922:web:0a6e09922375df4443bb8b",
  measurementId: "G-98J8L03MQW",
};

if (!firebase.apps.length) firebase.initializeApp(firebaseConfig);

export default {
  data() {
    return {
      title: "掲示板アプリ",
      message: "ログイン後、最新の投稿を表示します。",
      user_name: "",
      logined: false,
      msg: "",
      user: null,
      currentPage: 0,
      num_per_page: 10,
      json_data: [],
    };
  },
  methods: {
    login() {
      const provider = new firebase.auth.GoogleAuthProvider();
      firebase
        .auth()
        .signInWithPopup(provider)
        .then((result) => {
          console.log(result.user);
          this.user = result.user;
          this.user_name = result.user.displayName;
          this.message = "ログインしました。";
          this.logined = true;
          let db = firebase.database();
          let ref = db.ref("board");
          ref
            .orderByKey()
            .on("value", (snapshot) => {
              if (firebase.auth().currentUser != null) {
                const raw = snapshot.val() || {};
                const arr = [];
                for (const id in raw) {
                  const obj = raw[id];
                  arr.unshift({ id, ...obj }); // 時系列逆転
                }
                this.json_data = arr;
              } else {
                this.json_data = [];
              }
            });
        })
        .catch((err) => {
          this.message = err.message;
        });
    },
    logout() {
      firebase.auth().signOut();
      this.user_name = "";
      this.json_data = [];
      this.logined = false;
      this.message = "ログアウトしました。";
    },
    doLogin() {
      if (firebase.auth().currentUser == null) {
        this.login();
      } else {
        this.logout();
      }
    },
    add() {
      if (firebase.auth().currentUser == null) {
        alert("ログインしないと投稿できません。");
        return;
      }
      let user = firebase.auth().currentUser;
      let d = new Date();
      let dstr =
        d.getFullYear() +
        "-" +
        (d.getMonth() + 1) +
        "-" +
        d.getDate() +
        " " +
        d.getHours().toString().padStart(2, "0") +
        ":" +
        d.getMinutes().toString().padStart(2, "0") +
        ":" +
        d.getSeconds().toString().padStart(2, "0");
      let id = d.getTime();
      let data = {
        msg: this.msg,
        user: user.displayName,
        posted: dstr,
      };
      firebase
        .database()
        .ref("board/" + id)
        .set(data);
      this.msg = "";
      this.message = "投稿しました。";
    },
    deleteItem(id) {
      if (!firebase.auth().currentUser) {
        alert("ログインしないと削除できません。");
        return;
      }
      try {
        firebase.database().ref(`board/${id}`).remove();
        // 2) 画面側の配列からも除外
        this.json_data = this.json_data.filter((item) => item.id !== id);
        this.message = "投稿を削除しました。";
      } catch (err) {
        console.error(err);
        this.message = "削除に失敗しました：" + err.message;
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages - 1) this.currentPage++;
    },
    prevPage() {
      if (this.currentPage > 0) this.currentPage--;
    },
  },
  computed: {
    totalPages() {
      return Math.ceil(this.json_data.length / this.num_per_page);
    },
    pagedData() {
      const start = this.currentPage * this.num_per_page;
      return this.json_data.slice(start, start + this.num_per_page);
    },
  },
};
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
  font-size: 10px;
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
  align-items: center;
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
.board {
  position: relative;
  height: 500px;
  display: flex;
  flex-direction: column;
}
.pager {
  margin-top: auto;
}

.gsi-material-button {
  -moz-user-select: none;
  -webkit-user-select: none;
  -ms-user-select: none;
  -webkit-appearance: none;
  background-color: WHITE;
  background-image: none;
  border: 1px solid #747775;
  -webkit-border-radius: 4px;
  border-radius: 4px;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  color: #1f1f1f;
  cursor: pointer;
  font-family: "Roboto", arial, sans-serif;
  font-size: 14px;
  height: 40px;
  letter-spacing: 0.25px;
  outline: none;
  overflow: hidden;
  padding: 0 12px;
  position: relative;
  text-align: center;
  -webkit-transition: background-color 0.218s, border-color 0.218s,
    box-shadow 0.218s;
  transition: background-color 0.218s, border-color 0.218s, box-shadow 0.218s;
  vertical-align: middle;
  white-space: nowrap;
  width: auto;
  max-width: 400px;
  min-width: min-content;
}

.gsi-material-button .gsi-material-button-icon {
  height: 20px;
  margin-right: 12px;
  min-width: 20px;
  width: 20px;
}

.gsi-material-button .gsi-material-button-content-wrapper {
  -webkit-align-items: center;
  align-items: center;
  display: flex;
  -webkit-flex-direction: row;
  flex-direction: row;
  -webkit-flex-wrap: nowrap;
  flex-wrap: nowrap;
  height: 100%;
  justify-content: space-between;
  position: relative;
  width: 100%;
}

.gsi-material-button .gsi-material-button-contents {
  -webkit-flex-grow: 1;
  flex-grow: 1;
  font-family: "Roboto", arial, sans-serif;
  font-weight: 500;
  overflow: hidden;
  text-overflow: ellipsis;
  vertical-align: top;
}

.gsi-material-button .gsi-material-button-state {
  -webkit-transition: opacity 0.218s;
  transition: opacity 0.218s;
  bottom: 0;
  left: 0;
  opacity: 0;
  position: absolute;
  right: 0;
  top: 0;
}

.gsi-material-button:disabled {
  cursor: default;
  background-color: #ffffff61;
  border-color: #1f1f1f1f;
}

.gsi-material-button:disabled .gsi-material-button-contents {
  opacity: 38%;
}

.gsi-material-button:disabled .gsi-material-button-icon {
  opacity: 38%;
}

.gsi-material-button:not(:disabled):active .gsi-material-button-state,
.gsi-material-button:not(:disabled):focus .gsi-material-button-state {
  background-color: #303030;
  opacity: 12%;
}

.gsi-material-button:not(:disabled):hover {
  -webkit-box-shadow: 0 1px 2px 0 rgba(60, 64, 67, 0.3),
    0 1px 3px 1px rgba(60, 64, 67, 0.15);
  box-shadow: 0 1px 2px 0 rgba(60, 64, 67, 0.3),
    0 1px 3px 1px rgba(60, 64, 67, 0.15);
}

.gsi-material-button:not(:disabled):hover .gsi-material-button-state {
  background-color: #303030;
  opacity: 8%;
}
</style>
