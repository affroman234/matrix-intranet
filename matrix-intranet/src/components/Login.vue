<template> 
  <div class="is-three-quarters">
    <div id="google-signin-button" class="g-signin2 is-pulled-right" data-onsuccess="onSignIn">
      <meta name="google-signin-scope" content="email">
      <meta name="google-signin-client_id" content="XXXXXXXXX">
    </div>
  </div>
</template>

<script>
var AWS = require('aws-sdk')
//var gapi = require('gapi')

export default {
  name: 'Login',
  data () {
    return {
      loggedIn: false,
      idToken: null,
      isLoggedIn: false
    }
  }
}
</script>
