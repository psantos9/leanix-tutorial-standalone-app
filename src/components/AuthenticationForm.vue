<template>
  <div class="outer-container">
    <div class="inner-container">
      <div class="form-title">
        Authentication Form
      </div>
      <div class="input-group">
        <label for="instance">Instance</label>
        <br>
        <input id="instance" v-model="instance">
      </div>
      <div class="input-group">
        <label for="api-token">API token</label>
        <br>
        <input id="api-token" v-model="apiToken">
      </div>
      <button
        class="submit-button"
        :disabled="!instance || !apiToken"
        @click="fetchToken">
        <span v-if="!fetchingToken">
          Submit
        </span>
        <span v-else>
          <svg class="refresh-icon" fill="none" stroke="currentColor"  viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"  d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"></path>
          </svg>
        </span>
      </button>
      <div v-if="error" class="error-container">
        {{error}}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      instance: null,
      apiToken: null,
      error: null,
      fetchingToken: false
    }
  },
  methods: {
    async  fetchToken () {
      const { instance } = this
      const url = `https://${instance}/services/mtm/v1/oauth2/token`
      try {
        this.fetchingToken = true
        const requestOptions = {
          method: 'post',
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            Authorization: `Basic ${btoa(`apitoken:${this.apiToken}`)}`
          },
          body: 'grant_type=client_credentials'
        }
        const { access_token: accessToken = null } = await fetch(url, requestOptions)
          .then(response => {
            if (response.ok) return response.json()
            else throw Error(`${response.status}  ${response.statusText}`)
          })
        this.$emit('instance', instance)
        this.$emit('access-token', accessToken)
      } catch (error) {
        this.error = error
      } finally {
        this.fetchingToken = false
      }
    }
  }
}
</script>

<style scoped>
  .outer-container {
    height: 100vh;
    width: 100vw;
    display: flex;
    align-items: center;
    justify-content: center;
    background: lightgray;
  }

  .inner-container {
    border: 1px solid grey;
    border-radius: 5px;
    display: flex;
    flex-flow: column;
    align-items: center;
    padding: 2rem;
    position: relative;
    background: white;
  }

  .form-title {
    font-size: 1.7rem;
    margin-bottom: 1rem;
  }

  .input-group {
    padding: 0.6rem 0;
  }

  .input-group > input {
    font-size: 1.1rem;
    padding: 0.5rem 0.8rem;
    border: 1px solid grey;
    border-radius: 5px;
  }

  .submit-button {
    margin: 1.8rem;
  }

  .refresh-icon {
    width: 1rem;
    height: 1rem;
    animation: spin-animation 2s infinite;
  }

  .error-container {
    position: absolute;
    bottom: 0;
    margin-bottom: 0.5rem;
    color: red;
  }

  @keyframes spin-animation {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(-359deg);
    }
  }
</style>
