<template>
  <react-wrapper
    style="width: 100vw; height: 100vh"
    @hook:mounted="addLogoutBtnToGraphiQLToolbar"
    :component="graphiQL"
    :fetcher="fetcher"
  />
</template>

<script>
import { ReactWrapper } from 'vuera'
import GraphiQL from 'graphiql'
import 'graphiql/graphiql.css'

export default {
  props: {
    instance: {
      type: String,
      required: true
    },
    accessToken: {
      type: String,
      required: true
    }
  },
  components: {
    ReactWrapper
  },
  data () {
    return {
      graphiQL: GraphiQL
    }
  },
  methods: {
    async fetcher (graphQLParams) {
      const { instance, accessToken } = this
      const httpEndpoint = `https://${instance}/services/pathfinder/v1/graphql`
      const headers = {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${accessToken}`
      }
      const options = {
        method: 'post',
        headers,
        body: JSON.stringify(graphQLParams)
      }
      return fetch(httpEndpoint, options)
        .then(response => response.json())
    },
    addLogoutBtnToGraphiQLToolbar () {
      const topBarEl = this.$el.getElementsByClassName('topBar')[0]
      const logoutBtn = document.createElement('button')
      logoutBtn.innerText = 'Logout'
      logoutBtn.classList.add('toolbar-button')
      logoutBtn.onclick = () => this.$emit('logout')
      topBarEl.append(logoutBtn)
    }
  }
}
</script>
