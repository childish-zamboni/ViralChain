<template>
  <div class="hello">
    <div>
      <h1>ViralChain <img height="80" src="../assets/logo.png"></img></h1>
      <p>A decentralized news network built on <a href="https://blockstack.org" target="_blank">Blockstack</a></p>
      <button class="btn btn-default bg-white" @click.prevent="signIn">Sign In With Blockstack</button>
    </div>
  </div>
</template>

<script>
import { userSession } from '../userSession'

export default {
  name: 'landing',
  methods: {
    signIn () {
      userSession.redirectToSignIn()
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.hello {
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
