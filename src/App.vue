<template>
  <div id="app">
    <div
      v-if="accessToken === null"
      class="token-input-container">
      <div class="form-field">
        <label class="form-field-title">
          Instance
        </label>
        <input
          class="form-field-input"
          v-model.trim="instance"/>
      </div>
      <div class="form-field">
        <label class="form-field-title">
          API Token
        </label>
        <input
          class="form-field-input"
          v-model.trim="apiToken"/>
      </div>
       <button
        @click="fetchToken"
        :disabled="!apiToken"
        class="submit-btn">
        <span v-if="!submitting">Submit</span>
        <span v-else>
          <svg class="refresh-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"></path>
          </svg>
        </span>
      </button>

      <div
        v-if="error"
        class="error-container">
        {{error}}
      </div>
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
      error: null,
      submitting: false
    }
  },
  methods: {
    async fetchToken () {
      // const apiToken = 'N9eBqdeJsy7aCpqnJQMGsnCVNR3G7rebBsbNn2k7'
      const url = `https://${this.instance}/services/mtm/v1/oauth2/token`
      try {
        this.submitting = true
        const { access_token: accessToken = null } = await fetch(url, {
          method: 'post',
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            authorization: `Basic ${btoa(`apitoken:${this.apiToken}`)}`
          },
          body: 'grant_type=client_credentials'
        }).then(response => {
          if (response.ok) return response.json()
          else throw Error(`${response.status} ${response.statusText}`)
        })
        this.accessToken = accessToken
      } catch (error) {
        this.error = error
      } finally {
        this.submitting = false
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
    apiToken () {
      this.error = null
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

.token-input-container {
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: lightgray;
  color: black;
}

.form-field {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 3rem;
  min-width: 650px;
}

.form-field-title {
  font-size: 1.3rem;
  font-weight: bold;
  margin-bottom: 0.3rem;
  text-transform: uppercase;
}

.form-field-input {
  font-size: 1rem;
  padding: 0.5rem 1rem;
  border: 1px solid gray;
  border-radius: 5px;
}

.submit-btn {
  font-size: 1rem;
  padding: 0.5rem 1rem;
  border: 1px solid gray;
  border-radius: 5px;
  width: 100px;
}

.refresh-icon {
  width: 1rem;
  height: 1rem;
  animation: spin-animation 2s infinite;
}

.error-container {
  margin-top: 2rem;
  color: red;
  font-weight: bold;
}

@keyframes spin-animation {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(359deg);
  }
}

</style>
