<template>
  <section class="container">
    <h1>{{ title }}</h1>
    <p>{{ message }}</p>
    <pre>[{{ now }}]</pre>
    <router-link to="/other">Go to other</router-link>
    <router-link to="/memo">Go to Memo</router-link>
  </section>
</template>

<script>
export default {
  data:function () {
    return {
      title:'Hello',
      message:'this is message',
      now:'wait...',
    };
  },
  created:function () {
    setInterval(()=> {
      var d = new Date();
      this.now = d.getHours()
      + ':' + d.getMinutes()
      + ':' + d.getSeconds();
    },100)
  }
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
