<template>
  <section class="container">
    <h1>{{ title }}</h1>
    <p>{{ message }}</p>
    <!-- <pre>[{{ now }}]</pre>
    <router-link to="/other">Go to other</router-link>
    <router-link to="/memo">Go to Memo</router-link> -->
  </section>
</template>

<script>
import firebase from 'firebase/compat/app'
import 'firebase/compat/auth'

export default {
  data() {
    return {
      title:'Auth',
      message:'this is message'
    };
  },
  created:function () {
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

    const provider = new firebase.auth.GoogleAuthProvider()
    firebase.auth().signInWithPopup(provider).then(result => {
      this.message = `${result.user.displayName}, ${result.user.email}`
    }).catch(err => { this.message = err.message })
  },
};
</script>

<style>
  .container {
    padding: 5px 10px;
  }
  h1 {
    font-size: 60px;
    color: #345980;
  }
  p {
    padding-top: 5px;
    font-size: 20px;
  }
  pre {
    padding:10px;
    font-size: 18px;
    background-color: #efefef;
  }
</style>
