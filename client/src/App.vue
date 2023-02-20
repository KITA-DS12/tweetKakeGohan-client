<template>
  <div id="app">
    <!-- 名前とメッセージの各 input タグの入力値は、 それぞれ inputName, inputMessage と同期されます -->
    <input type="text" v-model="inputTag" placeholder="タグ" />
    <br>
    <button @click="setTag">設定</button>
    <br>
    <input type="text" v-model="inputName" placeholder="名前" />
    <br>
    <input type="text" v-model="inputMessage" placeholder="メッセージ" />
    <br>
    <button @click="sendMessage">送信</button>
    <ul>
      <!-- messages に入っているメッセージをループして出力します -->
      <li v-for="message in messages" :key="message.id">
        <b>{{ message.user_name }} :</b> {{ message.message }}
      </li>
    </ul>
  </div>
</template>

<script lang="ts" setup>
import { onBeforeUnmount, onMounted, Ref, ref } from 'vue';
import { createClient } from "@supabase/supabase-js"

const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY,
)

const messages: Ref<Array<any>> = ref([])
const inputTag: Ref<String> = ref("")
const tag: Ref<String> = ref("")
const inputName: Ref<String> = ref("")
const inputMessage: Ref<string> = ref("")

const setTag = () => {
  tag.value = inputTag.value
  getMessages()
}

const sendMessage = async () => {
  await supabase
    .from('Messages')
    .insert(
      {
        tag: inputTag.value,
        message: inputMessage.value,
        user_name: inputName.value,
      },
    )
  inputMessage.value = ""

}
const getMessages = async () => {
  let { data } = await supabase
    .from('Messages')
    .select('*')
    // @ts-ignore
    .ilike("tag", tag.value)
    .order("created_at", { ascending: false })
  messages.value = data
}
const subscribeMessage = () => {
  supabase
    .channel("any")
    .on("postgres_changes", { event: "INSERT", schema: "public", table: "Messages" }, payload => {
      console.log(payload)
      messages.value = [payload.new, ...messages.value].slice(0, 100)
    })
    .subscribe()
}
const unsubscribeMessage = () => {
  supabase.removeAllChannels()
}

onMounted(() => {
  getMessages()
  subscribeMessage()
})

onBeforeUnmount(() => {
  unsubscribeMessage()
})
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

nav {
  padding: 30px;
}

nav a {
  font-weight: bold;
  color: #2c3e50;
}

nav a.router-link-exact-active {
  color: #42b983;
}
</style>
