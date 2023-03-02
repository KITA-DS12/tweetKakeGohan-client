<template>
  <v-list bg-color="#0F1112" style="color: white;">
    <div style="height: 30px;" />
    <v-divider></v-divider>
    <v-list-item>
      <template v-slot:prepend>
        <v-icon color="blue-grey-lighten-3" style="padding-bottom: 50px" size="64" icon="mdi-account-circle"></v-icon>
      </template>
      <v-list-item-title>
        <v-text-field v-model="inputName" label="名前" variant="plain" density="compact" single-line>
        </v-text-field>
      </v-list-item-title>
      <v-list-item-subtitle>
        <v-textarea v-model="inputMessage" label="いまどうしてる？" auto-grow variant="plain" rows="2" max-rows="2"
          density="compact" single-line>
        </v-textarea>
      </v-list-item-subtitle>
    </v-list-item>
    <v-btn rounded="pill" flat color="#374FA3" class="text-white" @click="sendMessage"
      style="position: absolute; bottom: 10%; right: 5%;" v-bind:disabled="!isAllowedToPush">
      <b>つぶやく</b>
    </v-btn>
  </v-list>
  <v-list bg-color="#0F1112" style="color: white;">
    <v-divider></v-divider>
    <template v-for="message in messages" :key="message.id">
      <div>
        <v-list-item style="margin-top: 5px; margin-bottom: 5px;">
          <template v-slot:prepend>
            <v-icon :color="getColor(message.user_name)" size="48" icon="mdi-account-circle" style=""></v-icon>
          </template>

          <v-list-item-title class="text-grey-lighten-3"><b>{{ message.user_name }}</b></v-list-item-title>

          <v-list-item-subtitle class="text-grey-lighten-2" style="padding-top: 5px;">
            <b>{{ message.message }}</b>
          </v-list-item-subtitle>
        </v-list-item>
        <v-divider></v-divider>
      </div>
    </template>
  </v-list>
  <v-btn style="position: fixed; bottom: 10%; right: 5%;" icon="mdi-arrow-up-bold-outline" class="pagetop"
    color="#374FA3" size="72" href="#">
  </v-btn>
</template>
<script lang="ts" setup>
import { onBeforeUnmount, onMounted, Ref, ref, computed } from 'vue';
import { createClient } from "@supabase/supabase-js"

const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY,
)

const url = new URL(window.location.href)
const params = url.searchParams

if (params.get("tag") == null) {
  location.href = "?tag=adv"
}

const messages: Ref<Array<any> | null> = ref([])
const tag: string | null = "#" + params.get("tag")
const names: Ref<Array<string>> = ref([])
const colorsOrigin: Array<string> = ["red-lighten-1", "orange-lighten-1", "yellow-lighten-2", "lime-lighten-1", "light-green-lighten-2", "green-lighten-1", "teal-lighten-2", "cyan-lighten-1", "blue-lighten-1", "deep-purple-lighten-3", "purple-lighten-2", "pink-lighten-3"]
const colors: Ref<Array<string>> = ref(["red-lighten-1", "orange-lighten-1", "yellow-lighten-2", "lime-lighten-1", "light-green-lighten-2", "green-lighten-1", "teal-lighten-2", "cyan-lighten-1", "blue-lighten-1", "deep-purple-lighten-3", "purple-lighten-2", "pink-lighten-3"])
const iconColors: Ref<Array<string>> = ref([])
const inputName: Ref<string> = ref("")
const inputMessage: Ref<string> = ref("")


const isAllowedToPush = computed(() => {
  if (inputMessage.value != '' && inputName.value != '') {
    return true;
  } else {
    return false;
  }
})

const getColor = (name: string) => {
  const index = names.value.indexOf(name)
  let color: string = "#FFF"
  if (index == -1) {
    names.value.push(name)
    color = colors.value.splice(Math.floor(Math.random() * colors.value.length), 1)[0]
    iconColors.value.push(color)
    if (!colors.value.length) {
      colors.value = colorsOrigin
    }
  } else {
    color = iconColors.value[index]
  }
  return color
}

const sendMessage = async () => {
  await supabase
    .from('Messages')
    .insert(
      {
        tag: tag,
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
    .ilike("tag", tag)
    .order("created_at", { ascending: false })
  messages.value = data
}
const subscribeMessage = () => {
  supabase
    .channel("any")
    .on("postgres_changes", { event: "INSERT", schema: "public", table: "Messages" }, payload => {
      console.log(payload)
      if (payload.new.tag == tag) {
        messages.value = [payload.new, ...messages.value].slice(0, 100)
      }
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
html {
  scroll-behavior: smooth;
}

.pagetop {
  height: 50px;
  width: 50px;
  position: fixed;
  right: 30px;
  bottom: 30px;
  background: #fff;
  border: solid 2px #000;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2;
}
</style>
