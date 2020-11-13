<template>
  <div id="app">
    <div v-if="accessToken === null">
      <label for="api-token">ApiToken N9eBqdeJsy7aCpqnJQMGsnCVNR3G7rebBsbNn2k7</label>
      <input v-model.lazy.trim="apiToken"/>
      <div v-if="error">{{error}}</div>
    </div>
    <react-wrapper
      v-else
      class="full-screen"
      @hook:mounted="addLogoutBtnToGraphiQLToolbar"
      :component="graphiQL"
      :fetcher="fetcher"
      ref="graphiql"
      />
  </div>
</template>

<script>
import { ReactWrapper } from 'vuera'
import GraphiQL from 'graphiql'
import 'graphiql/graphiql.css'

export default {
  name: 'App',
  components: {
    ReactWrapper
  },
  data () {
    return {
      instance: 'app.leanix.net',
      graphiQL: GraphiQL,
      apiToken: null,
      accessToken: null,
      response: null,
      error: null
    }
  },
  methods: {
    async fetchToken (apiToken) {
      // const apiToken = 'N9eBqdeJsy7aCpqnJQMGsnCVNR3G7rebBsbNn2k7'
      const url = `https://${this.instance}/services/mtm/v1/oauth2/token`
      try {
        const { access_token: accessToken = null } = await fetch(url, {
          method: 'post',
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            authorization: `Basic ${btoa(`apitoken:${apiToken}`)}`
          },
          body: 'grant_type=client_credentials'
        }).then(response => {
          if (response.ok) return response.json()
          else throw Error(`${response.status} ${response.statusText}`)
        })
        this.accessToken = accessToken
      } catch (error) {
        this.error = error
      }
    },
    removeToken () {
      this.accessToken = null
      sessionStorage.removeItem('lnx_token')
    },
    async fetcher (graphQLParams) {
      const httpEndpoint = `https://${this.instance}/services/pathfinder/v1/graphql`
      const headers = {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${this.accessToken}`
      }
      const options = {
        method: 'post',
        headers,
        body: JSON.stringify(graphQLParams)
      }
      return fetch(httpEndpoint, options)
        .then(response => response.json())
    },
    reset () {
      this.accessToken = null
      this.apiToken = null
    },
    // https://stackoverflow.com/questions/55079866/vue-js-is-there-a-way-to-know-when-the-component-instance-is-mounted
    addLogoutBtnToGraphiQLToolbar () {
      const topBarEl = this.$refs.graphiql.$el.getElementsByClassName('topBar')[0]
      const logoutBtn = document.createElement('button')
      logoutBtn.innerText = 'Logout'
      logoutBtn.classList.add('toolbar-button')
      logoutBtn.onclick = this.reset
      topBarEl.append(logoutBtn)
    }
  },
  watch: {
    apiToken (value = null) {
      if (value !== null) this.fetchToken(value)
    }
  }
}
</script>

<style>
body {
  margin: 0
}

.full-screen {
  height: 100vh;
  width: 100vw;
}
</style>
