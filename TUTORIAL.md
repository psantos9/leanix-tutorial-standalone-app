# Creating a standalone cross-platform client app for LeanIX
LeanIX Pathfinder is a very versatile tool when it comes to explore, analyze and manage your workspace data. However, and in very specific situations, the user may come up with a set of requirements that call out for custom development, either in the form of a Custom Report, or a project of some sort that interfaces with any of the available LeanIX [APIs](https://dev.leanix.net/docs/available-apis).

In this tutorial, we'll guide you on how to build a simple cross-platform desktop app, based on [Vue](https://vuejs.org/) and [Electron](https://www.electronjs.org/), that allows the user to securely access the LeanIX Pathfinder [GraphQL API](https://www.electronjs.org/) using the [GraphiQL](https://github.com/graphql/graphiql) library.
<div  style="display:flex; justify-content:center">
  <img  src="https://i.imgur.com/zFJFFkB.png">
</div>

The complete source-code for this project can be found [here](https://github.com/pauloramires/leanix-tutorial-standalone-app).

## Pre-requisites

*  [NodeJS LTS](https://nodejs.org/en/) installed in your computer.

## Getting started
We'll build this project using mainly two Javascript frameworks: [Vue](https://vuejs.org/) and [Electron](https://www.electronjs.org/). In a nutshell, we'll use Vue to design our user interface and implement the business logic that interfaces with LeanIX APIs, and Electron for converting it into a distributable cross-platform application that runs directly on the user's machine, without need to be hosted on a server.

We'll start by bootstraping our project using the [@vue/cli](https://cli.vuejs.org/guide/) tool. Install this tool in your computer using the following command:
```bash
npm install -g @vue/cli
# OR
yarn global add @vue/cli
```

Once installed, create a new project as follows:

```bash
vue create leanix-tutorial-standalone-app
```
During the project creation, you will be prompted to pick a preset. You can simply choose the default preset which comes with a basic Babel + ESLint setup. Once all dependencies are installed, open the project folder  with your favorite code editor and  launch an instance of your development server by running the following command:

```bash
cd leanix-tutorial-standalone-app
npm run serve
# OR
yarn serve
```
Once the project compiles successfully, you should the following message on your terminal:
<div  style="display:flex; justify-content:center">
  <img  src="https://i.imgur.com/POy17nL.png">
</div>

You can now open a browser pointing to the local server address, http://localhost:8080 in this case. You should see the following output:
<div  style="display:flex; justify-content:center">
  <img  src="https://i.imgur.com/STlNRfn.png">
</div>

If by any reason you are not able to see the output in your browser, then review the preview steps. Otherwise, let's continue!

Without touching our source code yet, we'll add the [electron-builder](https://github.com/nklayman/vue-cli-plugin-electron-builder) plugin to our project. This plugin installs all the required boilerplate code and dependencies required to transform our Vue project into an electron app. To add the plugin simply run the following command in your project's directory:
```bash
vue add electron-builder
```
You’ll be prompted to select your preferred Electron version, just choose the latest version that is listed (`9.0.0` at the time of writing).
When this is done, you can launch your electron application using the following command:
```bash
npm run electron:serve
# OR
yarn electron:serve
```
It will take a time to compile and serve your app. Once the project launches, you should be able to see a new window popping up in your desktop like to one below:
<div style="display:flex; justify-content:center">
  <img src="https://i.imgur.com/ZDaYWri.png">
</div>

If you close the developer tools tab, it will look like this:
<div style="display:flex; justify-content:center">
  <img src="https://i.imgur.com/QRGUvia.png">
</div>

The advantage of having this electron plugin approach is that you can develop your app as a regular Vue application, and in the end compile it into a standalone application, sharing exactly the same codebase.

 Now, and before actually starting the implementation of our project, we make the final adjustments in our boilerplate codebase. We delete the file *components/HelloWorld.vue* and set the contents of *src/App.vue* file as follows:
```html
<template>
  <div id="app">
  </div>
 </template>

<script>
  export default {
    name: 'App'
  }
</script>

<style>
  body {
    margin: 0;
  }
</style>
```
There you go, we are now good to start coding our project!



## Application workflow and design

We'll build our application around two main views:
1. A form in which the user inputs the authentication credentials (leanix instance and api token)
2. The graphiql user interface

The first view will consist of a login form with two input fields. The first input is used to specify the LeanIX instance in which his workspace resides. The second input is used to set the API key to be used for fetching an access token. Additionally, a submit button and a field for displaying eventual authentication errors will be included as well. The submit button is disabled until both the instance and api key inputs are filled. Once the user clicks on the submit button, the application will query the LeanIX authentication endpoint in order to fetch an access token. If a valid instance/api token combination is submitted to the server, then the user receives an access token which can be used to query the LeanIX GraphQL API. On the other hand, if the user submits an invalid combination, then a 403 Unauthorized error is displayed to the user.

Once a valid access token is received by the application, the view switches to the standard [GraphiQL](https://github.com/graphql/graphiql) user interface, which can be used to query the LeanIX GraphQL API. Since we are implementing the log in/log out workflow for authenticating the application, we'll make a small customization to the GraphiQL user interface for adding a logout button.

### Implementing the application authentication form (view 1)

Create a new file component *src/components/AuthenticationForm.vue* as follows:
```html
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
```
Once we have our *AuthenticationForm* component set, we'll integrate it into our app by setting the *src/App.vue* file as follows:
```html
<template>
  <div id="app">
    <!-- render the authentication-form component only when the accessToken is not defined -->
    <!-- notice the 'instance' and 'access-token' event listeners that set the instance and accessToken state variables, respectively -->
    <authentication-form
      v-if="accessToken === null"
      @instance="instance = $event"
      @access-token="accessToken = $event"/>
    <div v-else>Authenticated!</div>
  </div>
</template>

<script>
import AuthenticationForm from '@/components/AuthenticationForm'
export  default {
  name: 'App',
  components: {
    AuthenticationForm
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
```
Now, and if necessary, launch again your project with the following command:

```bash
npm run electron:serve
# OR
yarn electron:serve
```

You should see the AuthenticationForm popping up in your desktop!
<div style="display:flex; justify-content:center">
  <img src="https://i.imgur.com/baakZ44.png">
</div>

Before proceeding, you will need a valid [LeanIX API Token](https://docs.leanix.net/docs/create-and-manage-your-own-api-tokens). Once you have it, fill in the **instance** and **API token** fields of your input form and click on the **Submit** button. You should get the following [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) error marked in red:

<div style="display:flex; justify-content:center">
  <img src="https://i.imgur.com/ekXK7yX.png">
</div>

Since our application is running locally on the user machine and not being hosten on a webserver, we can safely overcome this issue by opening the *src/background.js* file and set the **webSecurity**  flag to true in the **webPreferences** object (line 22) as in the picture below:
<div style="display:flex; justify-content:center">
  <img src="https://i.imgur.com/RhsSWNh.png">
</div>

Once the flag is set, relaunch your project and submit again your authentication credentials. You should be able now to sucessfully get an access token and see the following message:
<div style="display:flex; justify-content:center">
  <img src="https://i.imgur.com/n99KSd7.png">
</div>

If that's the case, then good job! If not, we suggest to review all the previous steps of this tutorial and compare them with your project for possible differences.

### Implementing the customized GraphiQL user interface (view 2)
As mentioned previously, the second view of our project will be based on the [GraphiQL](https://github.com/graphql/graphiql) library. However, and since we are developing a Vue project and GraphiQL is based on [React](https://en.reactjs.org/), we need to wrap it using an auxiliary library - [vuera](https://github.com/akxcv/vuera), to make it run seamlessly inside our project. Therefore,  we start by installing the following dependencies
```bash
npm install graphiql vuera graphql prop-types react react-dom
# OR
yarn add graphiql vuera graphql prop-types react react-dom
```
Once the dependencies are installed,  we create the *src/component/GraphiQL.vue* component as follows:
```html
<template>
  <!-- wrapper for the React GraphiQL component -->
  <!-- notice the 'mounted' lifecycle hook that triggers the addLogoutBtnToGraphiQLToolbar method -->
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
  // instance and accessTokens are props received from the parent component
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
    // method responsible for querying the LeanIX GraphQL API, used by the GraphiQL component
    async fetcher (graphQLParams) {
      const { instance, accessToken } = this
      const httpEndpoint = `https://${instance}/services/pathfinder/v1/graphql`
      const headers = {
        'Content-Type':  'application/json',
        Authorization:  `Bearer ${accessToken}`
      }
      const options = {
        method: 'post',
        headers,
        body: JSON.stringify(graphQLParams)
      }
      return fetch(httpEndpoint, options)
        .then(response => response.json())
    },
    // Method that adds the 'Logout' button to the GraphiQL toolbar and triggers
    // a 'logout' event to the parent component whenever the user clicks on it
    addLogoutBtnToGraphiQLToolbar () {
      const topBarEl = this.$el.getElementsByClassName('topBar')[0]
      const logoutBtn = document.createElement('button')
      logoutBtn.innerText = 'Logout'
      logoutBtn.classList.add('toolbar-button')
      logoutBtn.onclick = () =>  this.$emit('logout')
      topBarEl.append(logoutBtn)
    }
  }
}
</script>
```
Having set our GraphiQL component, we'll integrate it into our application by editing the *src/App.vue* file as follows:
```html
<template>
  <div id="app">
    <!-- authentication-form shown when not logged in -->
    <authentication-form
      v-if="accessToken === null"
      @instance="instance = $event"
      @access-token="accessToken = $event"/>
    <!-- when logged in show the graphiql component -->
    <!-- we pass the 'instance' and 'accessToken' here as props -->
    <!-- and notice the event listener for the 'logout' events that clears them -->
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
export  default {
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
```
Now again, and if not running, relaunch your project with the following command:

```bash
npm run electron:serve
#OR
yarn electron:serve
```
Fill in the authentication form with your instance and Api token and submit it. You should be able to see the GraphiQL interface once the authentication process completes. Notice as well the **Logout** button located on the GraphiQL toolbar. Now make some queries to the LeanIX GraphiQL API and make sure everything works as expected.
<div style="display:flex; justify-content:center">
  <img src="https://i.imgur.com/MUlfiyy.png">
</div>

Finally, click on the **Logout** button and make sure the application returns to the Authorization Form view.
<div style="display:flex; justify-content:center">
  <img src="https://i.imgur.com/3IDawV0.jpg">
</div>

If everything went as expected, congratulations for having finished this tutorial!
Otherwise, please review your steps for eventual differences between your project and the tutorial.

## Building up our application
Once we have complete the implementation of our source code, and in order to distribute our application to the user, we need to build it using the following command:
```bash
npm run electron:build
#OR
yarn electron:build
```
By default, this command will bundle the application into a single executable file and place it into the *dist* folder in your project.
The [vue-cli-plugin-electron-builder](https://nklayman.github.io/vue-cli-plugin-electron-builder/) that is included in this project uses the [electron-builder](https://www.electron.build/) library. You require additional information about building executables for other architectures than the one you are using on your development machine, please refer to the electron build [documentation](https://www.electron.build/cli).

And that's it, you now have a distributable stand-alone client app that can interface with LeanIX GraphQL API. Congratulations on finishing this tutorial!
