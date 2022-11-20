<template>
  <div class="root">
    <section>
      <form>
        <select @click="loadHistory($event.target.value)">
          <option v-for="(h, i) in history" :value="i">{{ h.apiUrl }}</option>
        </select>

        <div>
          <select v-model="requestType">
            <option value="post">POST</option>
            <option value="get">GET</option>
            <option value="put">PUT</option>
            <option value="delete">DELETE</option>
            <option value="patch">PATCH</option>

          </select>
          <input class="url-field" type="text" placeholder="api url" v-model="apiUrl" />
          <button @click="this.execute">Execute</button>

        </div>

        <div class="actions">
          <button type="button" @click="selectedTab = 'headers'"
            :class="selectedTab === 'headers' && 'active'">headers</button>
          <button type="button" @click="selectedTab = 'params'"
            :class="selectedTab === 'params' && 'active'">params</button>
          <button type="button" @click="selectedTab = 'body'" :class="selectedTab === 'body' && 'active'">body</button>
        </div>

        <p>{{ selectedTab }}</p>


        <textarea v-if="selectedTab === 'body'" placeholder="body(json)" v-model="body" />



        <div v-if="selectedTab === 'headers'" v-for="(header, index) in headers">
          <input type="text" placeholder="key" v-model="header.key" />
          <input type="text" placeholder="value" v-model="header.value" />
          <button v-if="index === headers.length - 1 && header.key && header.value" type="button"
            @click="headers = [...headers, { key: '', value: '' }]">Add New</button>
        </div>

        <div v-if="selectedTab === 'params'" v-for="(param, index) in params">
          <input type="text" placeholder="key" v-model="param.key" />
          <input type="text" placeholder="value" v-model="param.value" />
          <button v-if="index === params.length - 1 && param.key && param.value" type="button"
            @click="params = [...params, { key: '', value: '' }]">Add New</button>
        </div>
      </form>

    </section>

    <section>
      <p v-if="responseCode">Response Code:{{ responseCode }}</p>

      <p>Response</p>

      <pre>{{ response }}</pre>

    </section>


  </div>

</template>

<script>
import axios from "axios";
import VueJsonPretty from 'vue-json-pretty';
import 'vue-json-pretty/lib/styles.css';

export default {
  name: 'app',
  components: {
    VueJsonPretty
  },
  methods: {

    execute: function (e) {
      e.preventDefault()
      const reqHeaders = { 'Content-Type': 'application/json;charset=UTF-8' }
      for (let header of this.headers) {
        if (header.key.trim() && header.value.trim()) {
          reqHeaders[header.key] = header.value
        }
      }

      let params = ""
      for (let param of this.params) {
        if (param.key.trim() && param.value.trim()) {
          console.log("d")
          params += `&${param.key}=${param.value}`
        }
      }
      params = params.replace("&", "?")

      //saving the history in local storage
      const newHistory = {
        apiUrl: this.apiUrl,
        params: this.params,
        headers: this.headers,
        body: this.body,
        requestType: this.requestType,
      }
      const updatedHistory = [...this.history, newHistory]
      localStorage.setItem("history", JSON.stringify(updatedHistory))

      axios[this.requestType](this.apiUrl + params, this.requestType === "get" ? { headers: reqHeaders } : this.body ? this.body : {}, this.requestType !== "get" ? { headers: reqHeaders } : {}).then(res => {
        this.response = res.data
        this.responseCode = 200

      }).catch(err => {
        this.responseCode = err.response.status
        this.response = ""
      }
      )
    },
    loadHistory: function (index) {
      const selectedHistory = this.history[index]
      this.apiUrl = selectedHistory.apiUrl
      this.params = selectedHistory.params
      this.headers = selectedHistory.headers
      this.body = selectedHistory.body
      this.requestType = selectedHistory.requestType
    }
  },
  watch: {

  },
  data() {
    return {
      apiUrl: "",
      requestType: "get",
      headers: [{ key: "Content-Type", value: "application/json" }],
      params: [{ key: "", value: "" }],
      body: "",
      response: "",
      selectedTab: "params",
      responseCode: '',
      history: JSON.parse(localStorage.getItem("history") || "[]")
    }
  }
}


</script>



<style>
@import url('https://fonts.googleapis.com/css2?family=PT+Sans&display=swap');


body {
  padding: 0;
  margin: 0;
  font-family: "PT Sans";
}

button:hover {
  cursor: pointer;
}

.actions {
  display: flex;
  align-items: center;
  gap: 10px;
}

.actions button {
  border: none;
  text-align: left;
  padding: 5px;
  border-radius: 0px;
}

.active {
  border-bottom: solid 1px coral !important;
}

.root form {
  display: flex;
  flex-direction: column;
  width: 500px;
  margin: auto;
  gap: 10px;
  justify-content: center;
  padding-top: 50px;
}

.root {
  background: black;
  min-height: 100vh;
  color: white;
  display: flex;
  justify-content: center;
  gap: 25px;
  width: 100%;
}

textarea {
  font-family: inherit;
  padding: 5px;
  width: 100%;
  max-width: 100%;
  min-width: 100%;
  height: 200px;
  background: black;
  outline: none;
  color: white;
}

input,
select,
button {
  padding: 5px 15px 5px 5px;
  outline: none;
  background: black;
  color: white;
  border: solid 1px gray;
  font-family: inherit;
  border-radius: 3px;
}

.url-field {
  min-width: 300px;
}

pre {
  max-height: 60vh;
  overflow-y: auto;
}

section {
  width: 50%;
}

section:first-child {
  border-right: solid 0.000005px gray;
}

section:last-child {
  margin-top: 124px;
}


select {
  min-width: 50px;
  font-size: 14px;
  border-radius: 3px;
  background-color: none;
  color: white;
  -webkit-appearance: none;
  -moz-appearance: none;
  text-indent: 1px;
}
</style>
