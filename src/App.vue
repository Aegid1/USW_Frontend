<template>
  <div class="d-flex align-items-center flex-shrink-0 p-3 link-body-emphasis text-decoration-none border-bottom">
    <img src="@/assets/logo.png" height="50" />
  </div>
  <div class="container">
    <div class="d-flex flex-column align-items-stretch flex-shrink-0 bg-body-tertiary">
      <div class="list-group list-group-flush scrollarea">
        <div v-if="messages.length === 0" class="list-group-item" id="welcome">
          <h2>Welcome to the media compass!</h2>
          <p>The media compass has access to a vector database which contains various articles
            from within the last few years to the above-mentioned topics. It will iterate through all the data and will
            answer your question based on the information in the database.</p>
        </div>
        <div class="list-group-item list-group-item-action py-3 lh-sm"
          v-for="message in messages" :key="message.id"
        >
          <div class="d-flex w-100 align-items-center justify-content-between">
            <strong class="mb-1">{{message.username}}</strong>
          </div>
          <div class="col-10 mb-1 small">{{message.message}}</div>
          <div v-if="message.visualization_given" class="graph-container">
            <!-- TODO: Testen & Styling -->
            <iframe src="http://localhost:8501" width="100%" height="400"></iframe>
          </div>
        </div>
        <div v-if="fetchRun" class="list-group-item spinnercustom"><OrbitSpinner :size="55" color="#f0f8ff" /></div>
      </div>
    </div>
    <form @submit.prevent="submit">
      <div class="input-group">
        <input class="form-control" placeholder="Write a message" v-model="message"/>
        <div class="input-group-append submit-div">
          <button class="btn btn-outline-secondary" type="submit" id="submit-icon">
            <BIconSend></BIconSend>
          </button>
        </div>
      </div>
    </form>
  </div>

  <div class="credits">
    <p>Ägid Haslauer, Mohamed Moussa, Jakob Scharf</p>
  </div>
</template>

<script>
import {BIconSend} from "bootstrap-icons-vue";
import {OrbitSpinner} from "epic-spinners";

export default {
  name: 'App',
  components: {
    BIconSend,
    OrbitSpinner
  },
  data() {
    return {
      messages: [],
      message: "",
      fetchRun: false,
      thread_id: "",
      visualization_given: false,
    }
  },
  methods: {
    async submit() {

      this.messages.push({
        id: Math.random().toString(36),
        username: "You",
        message: this.message,
        visualization_given: this.visualization_given,
      })


      if(this.messages.length === 1){
        this.fetchRun = true;
        console.log("FIRST SUBMIT")
        try {
          const response = await fetch("http://localhost:4000/api/v1/chat", {method: "POST"})
          const data = await response.json();
          this.thread_id = data.id;
          console.log("THREAD ID: ", this.thread_id);
          this.fetchRun = false;
        } catch(e) {
          console.error("Error creating thread: ", e)
        }
      }

      try {
        console.log("SENDING MESSAGE: ", this.message)
        this.fetchRun = true

        const response = await fetch("http://localhost:4000/api/v1/chat/" + this.thread_id, {
          method: "POST",
          headers: {'Content-Type': 'application/json'},
          body: JSON.stringify( {text: this.message})
        });
        const data = await response.json();
        this.messages.push({
          id: data.id,
          username: "media compass",
          message: data.text,
          visualization_given: data.visualization_given
        })
        console.log("RECEIVED MESSAGE FROM ", data)
        this.fetchRun = false
      } catch(e) {
        console.log("Error sending message: ", e)
        this.fetchRun = false
      }
      this.message = ''
    }
  },
}
</script>

<style>
  body {
    background-color: #121212;
    font-family: 'Jost', sans-serif;
    margin: 0;
  }
  h1,h2,h3,h4,h5,h6,span {
    color: #E7E6E5 !important;
  }
  input {
    background-color: #121212!important;
    color: #fff!important;
  }
  h1 {
    font-size: 30px;
    font-weight: 800;
  }
  .scrollarea {
    min-height: 500px;
  }
  .border-bottom {
    border-bottom: 1px solid aliceblue !important;
  }
  .credits {
    position: absolute;
    bottom: 0;
    right: 0;
    padding: 5px;
    color: gray;
  }
  .spinnercustom {
    display: flex;
    justify-content: center;
    align-items: center;
  }
  #welcome {
    text-align: center;
    margin-top: 4rem;
  }
  .list-group-item {
    background-color: #121212;
    color: aliceblue;
  }
  .btn-outline-secondary {
    border-color: #E7E6E5;
  }
  .btn-outline-secondary:hover {
    background-color: #E7E6E5 !important;
    border-color: #E7E6E5;
  }
  .btn-outline-secondary:hover svg path {
    fill: #121212;
  }
  svg {
    color: #E7E6E5;
  }


  /* jost-regular - latin */
  @font-face {
    font-display: swap; /* Check https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display for other options. */
    font-family: 'Jost';
    font-style: normal;
    font-weight: 400;
    src: url('./assets/jost-v15-latin-regular.woff2') format('woff2'); /* Chrome 36+, Opera 23+, Firefox 39+, Safari 12+, iOS 10+ */
  }
  /* jost-600 - latin */
  @font-face {
    font-display: swap; /* Check https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display for other options. */
    font-family: 'Jost';
    font-style: normal;
    font-weight: 600;
    src: url('./assets/jost-v15-latin-600.woff2') format('woff2'); /* Chrome 36+, Opera 23+, Firefox 39+, Safari 12+, iOS 10+ */
  }
  /* jost-800 - latin */
  @font-face {
    font-display: swap; /* Check https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display for other options. */
    font-family: 'Jost';
    font-style: normal;
    font-weight: 800;
    src: url('./assets/jost-v15-latin-800.woff2') format('woff2'); /* Chrome 36+, Opera 23+, Firefox 39+, Safari 12+, iOS 10+ */
  }
</style>
