<template>
  <div class="chat-container">
    <label class="switch">
      CHAT<input v-model="model" type="checkbox" checked="">IMAGE
    </label>
    <div class="chat-box" ref="botLoading">
      <ul class="message-list">
        <li v-for="(message, index) in messages" :key="index"
          :class="{ 'user-message animate__animated animate__rubberBand': message.role === 'user', 'bot-message': message.role === 'assistant' || message.role === 'error' || message.role === 'image' }"
          v-show="message.role != 'system'">
          <div class="content" v-if="message.role === 'assistant' || message.role === 'error' || message.role === 'user'"
            :style="{ 'color': message.role === 'error' ? 'brown' : '#333' }">
            {{ message.content }}
          </div>
          <div class="content" v-else-if="message.role === 'image'"
            :style="{ 'color': message.role === 'error' ? 'brown' : '#333' }">
            <img :src="message.content" />
          </div>
          <div class="avatar">
            {{ message.role === 'user' ? 'YOU' : 'BOT' }}
          </div>
        </li>
        <li v-show="loading" class="bot-message animate__animated animate__rubberBand	">
          <div class="content"><lottie-player src="https://assets6.lottiefiles.com/packages/lf20_vvx2gjpt.json"
              background="transparent" speed="1" style="width: 50px; height: 50px;" loop autoplay></lottie-player></div>
          <div class="avatar">
            {{ 'BOT' }}
          </div>
        </li>
      </ul>
    </div>
    <div class="input-box">
      <input class="user-input" v-model="userInput" @keyup.enter="keyup" type="text"
        placeholder="Waiting for your message ..." autofocus />
      <button class="send-button" @click="keyup" :disabled="loading">{{ loading ? '...' : 'Send'
      }}</button>
    </div>
    <div class="api-key-box">
      <label for="api-key-input">Enter your ChatGPT3.5 API Key:</label>
      <input id="api-key-input" v-model="apiKey" type="text" placeholder="Your API Key" />
    </div>
  </div>
</template>

<script setup>
import { nextTick, ref, watch } from 'vue';
import 'animate.css';
import axios from 'axios';
import { Configuration, OpenAIApi } from 'openai'
const apiKey = ref('');
const userInput = ref(null);
const messages = ref([]);
const loading = ref(false)
const setting = ref('在回答最后加上符合回答的emoji')
const timer = ref()
const botLoading = ref(null)
const lastLi = ref(null)
const model = ref(false)
messages.value.push({ role: 'system', content: setting.value })
watch(model, async () => {
  clearAll()
})
function clearAll () {
  // userInput.value = ''
  messages.value = []
  messages.value.push({ role: 'system', content: setting.value })
}
function keyup () {

  if (userInput.value) {
    if (model.value) {
      clearAll()
    }
    let errorIndex = messages.value.findIndex(item => item.role == 'error')
    if (errorIndex != -1) {
      messages.value.splice(errorIndex - 1, 2)
    }
    messages.value.push({ role: 'user', content: userInput.value })
    nextTick(() =>
      scrollToBottom()
    )
    if (model.value) {
      createImage()
    } else {
      timer.value = setTimeout(() => sendMessage(), 500)

    }

  }
}
function scrollToBottom () {
  botLoading.value.scrollTop = botLoading.value.scrollHeight - botLoading.value.clientHeight;
}
async function createImage () {

  if (userInput.value) {
    loading.value = true
    nextTick(() =>
      //console.log(lastLi.value)
      scrollToBottom()
    )
    const configuration = new Configuration({
      apiKey: apiKey.value,
    });
    const openai = new OpenAIApi(configuration);
    await openai.createImage({
      prompt: userInput.value,
      n: 1,
      size: "1024x1024",
    }).then(res => {
      console.log(res)
      nextTick(() =>
        //console.log(lastLi.value)
        scrollToBottom()
      )
      messages.value.push({ role: 'image', content: res.data.data[0].url })
      loading.value = false
    }).catch(error => {
      timer.value = setTimeout(() => {
        messages.value.push({ role: 'error', content: error.message })
        loading.value = false
      }
        , 1000)
      console.log(error)
    });
    timer.value = null
  }

}
async function sendMessage () {
  if (userInput.value) {

    loading.value = true
    nextTick(() =>
      scrollToBottom()
    )
    // const configuration = new Configuration({
    //   apiKey: apiKey.value,
    // });
    // const openai = new OpenAIApi(configuration);
    // const completion = await openai.createChatCompletion({
    //   model: "gpt-3.5-turbo",
    //   messages: messages.value,
    //   max_tokens: 1000,
    //   temperature: 0.5
    // }).then(res => {

    //   messages.value.push(res.data.choices[0].message)
    //   loading.value = false
    //   nextTick(() =>
    //     //console.log(lastLi.value)
    //     scrollToBottom()
    //   )


    //   userInput.value = null
    // }).catch(error => {
    //   timer.value = setTimeout(() => {
    //     messages.value.push({ role: 'error', content: error.message })
    //     loading.value = false
    //   }
    //     , 1000)
    //   console.log(error)
    // });
    axios.post('/v1/chat/completions', {
      model: "gpt-3.5-turbo",
      messages: messages.value,
      max_tokens: 1000,
      temperature: 0.5
    }, {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + apiKey.value
      }
    }).then(res => {

      messages.value.push(res.data.choices[0].message)
      loading.value = false
      nextTick(() =>
        //console.log(lastLi.value)
        scrollToBottom()
      )


      userInput.value = null
    }).catch(error => {
      timer.value = setTimeout(() => {
        messages.value.push({ role: 'error', content: error.message })
        loading.value = false
      }
        , 1000)
      console.log(error)
    });
    timer.value = null
  }
}





</script>

<style scoped>
/* your styles here */
@media (max-width: 768px) {
  .chat-box {
    padding: 20px !important;
  }

  img {
    width: 240px;
    height: 240px;
  }
}

img {
  width: 300px;
  height: 300px
}

.chat-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  justify-content: flex-end;
  background: #e8e8e8;
}

.chat-box {
  height: 80vh;
  overflow-y: scroll;
  padding: 20px 10%;
  box-sizing: border-box;
}

.message-list {
  list-style-type: none;
  margin: 0;
  padding: 0;
  width: 100%;
  display: flex;
  flex-direction: column;
  box-sizing: border-box;

}

.message-list .user-message {
  margin-top: 40px;
  display: flex;
  align-items: flex-start;
}

.message-list .bot-message {
  margin-top: 40px;
  display: flex;
  align-items: flex-start;
  flex-direction: row-reverse;
}

.message-list li .content {
  max-width: 50vw;
  /* height: 254px; */
  background: #fff;
  border: solid 4px #333;
  transition: all .2s ease-in-out;
  display: flex;
  cursor: pointer;
  color: #333;
  border-radius: 10px;
}

.user-message {
  align-self: flex-end;
  /* margin-left: 30vh; */
}

.user-message .content {
  text-align: left;
  box-shadow: 5px 5px #333;
  padding: 20px 20px 40px 40px;

  /* margin-left: 30vh; */
}

.bot-message {
  align-self: start;

}

.bot-message .content {
  text-align: left;
  box-shadow: -5px 5px #333;
  padding: 20px 40px 40px 20px;

}

.user-message::after {
  /* content: " <"; */
  color: #0099ff;
}

.bot-message::before {
  /* content: "> "; */
  color: #ff9900;
}

.input-box {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 20px;

}

.user-input {
  /* -webkit-appearance: none; */
  padding: 10px;
  width: 70%;
  font-size: 1.2rem;
  background: #fff;
  border: solid 4px #333;
  display: flex;
  cursor: pointer;
  -webkit-box-shadow: 5px 5px #333;
  box-shadow: 5px 5px #333;
  border-radius: 10px;
}

.user-message .avatar {
  box-shadow: 5px 5px #333;
  margin-left: 10px;
}

.bot-message .avatar {
  box-shadow: -5px 5px #333;
  margin-right: 10px;

}

.avatar {
  padding: 4px;
  width: 36px;
  height: 36px;
  font-size: 0.6rem;
  /* vertical-align: middle; */
  font-weight: bolder;
  text-align: end;
  background: #fff;
  border: solid 4px #333;
  transition: all .2s ease-in-out;
  display: flex;
  cursor: pointer;
  border-radius: 10px;
  justify-content: center;
  align-items: center;
}

.user-input:focus {
  outline: none;
}

.send-button {
  padding: 10px;
  font-size: 1.2rem;

  background: #333;
  border: solid 4px #fff;
  transition: all .2s ease-in-out;
  display: flex;
  cursor: pointer;
  box-shadow: 5px 5px #fff;
  border-radius: 10px;
  color: #fff;
  margin-left: 20px;
}

.api-key-box {
  margin: 20px;
}

label {
  font-size: 1rem;
  margin-right: 10px;
}

.switch {
  --false: #333;
  --true: #333;
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: bolder;
}

input[type=checkbox] {
  appearance: none;
  height: 2rem;
  width: 4.5rem;
  background-color: #fff;
  position: relative;
  border-radius: 10px;
  cursor: pointer;
  box-shadow: 5px 5px #333;
  border: solid 4px #333;
  padding: 16px;
  margin: 10px;
}

input[type=checkbox]::before {
  content: '';
  display: block;
  height: 1.9em;
  width: 1.9em;
  transform: translate(-50%, -50%);
  position: absolute;
  top: 50%;
  left: calc(1.9em/2 + .3em);
  background-color: var(--false);
  border-radius: .2em;
  transition: .3s ease;
}

input[type=checkbox]:checked::before {
  background-color: var(--true);
  left: calc(100% - (1.9em/2 + .3em));
}
</style>
