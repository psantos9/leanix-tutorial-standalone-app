<template>
  <div id="app">
    <authentication-form
      v-if="accessToken === null"
      @instance="instance = $event"
      @access-token="accessToken = $event"/>
    <graphi-ql
      v-else
      :instance="instance"
      :access-token="accessToken"
      @logout="instance = null; accessToken = null"
    />
  </div>
</template>

<script>
import AuthenticationForm from '@/components/AuthenticationForm'
import GraphiQl from '@/components/GraphiQL'
// const apiToken = 'N9eBqdeJsy7aCpqnJQMGsnCVNR3G7rebBsbNn2k7'
export default {
  name: 'App',
  components: {
    AuthenticationForm,
    GraphiQl
  },
  data () {
    return {
      instance: null,
      accessToken: null
    }
  }
}
</script>

<style>
body {
  margin: 0;
}
</style>
