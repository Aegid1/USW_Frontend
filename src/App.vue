<template>
  <div class="d-flex align-items-center flex-shrink-0 p-3 link-body-emphasis text-decoration-none border-bottom">
    <h1 class="fs-5">media compass</h1>
  </div>
  <div class="container">
    <div class="d-flex flex-column align-items-stretch flex-shrink-0 bg-body-tertiary">
      <div class="list-group list-group-flush scrollarea">
        <div v-if="messages.length === 0" class="list-group-item" id="welcome">
          <h2>Welcome to the media compass!</h2>
          <p>Your can ask the compass everything you want regarding the following topics:</p>
          <li>1. AfD</li>
          <li>2. Umwelt</li>
          <li>O.ä.</li>
          <p>The media compass has access to a vector database which contains various articles
            from within the last ten years to the above-mentioned topics. It will iterate through all the data and will
            answer your question based on the information in the database.</p>
        </div>
        <div class="list-group-item list-group-item-action py-3 lh-sm"
          v-for="message in messages" :key="message.id"
        >
          <div class="d-flex w-100 align-items-center justify-content-between">
            <strong class="mb-1">{{message.username}}</strong>
            <small class="text-body-secondary">Tues</small>
          </div>
          <div class="col-10 mb-1 small">{{message.message}}</div>
          <div v-if="message.graph" class="graph-container">
            <!-- TODO: Testen & Styling -->
            <iframe src="http://localhost:8501" width="100%" height="200"></iframe>
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
      fetchRun: false
    }
  },
  methods: {
    async submit() {
      try {
        this.fetchRun = true
        await fetch('http://localhost:8081/api/messages', {
          method: "POST",
          headers: {'Content-Type': 'application/json'},
          body: JSON.stringify({
            message: this.message.value
          })
        }).then(response => response.json())
            .then(data => {
              this.messages.push({
                id: data.id,
                username: data.username,
                message: data.message,
                graph: data.graph
              });
            })
      } catch(e) {
        console.log(e)
        this.fetchRun = false
      }
      this.message = ''
    }
  },
}
</script>

<style>
  body {
    background-color: #241D21;
    font-family: 'Jost', sans-serif;
    margin: 0;
  }
  h1,h2,h3,h4,h5,h6,span {
    color: #E7E6E5 !important;
  }
  input {
    background-color: #241D21!important;
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
    background-color: #241D21;
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
    fill: #241D21;
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
