<template>
  <div id="app">
    <v-row no-gutters>
      <v-sheet class="d-none d-md-block" color="#0F1112" width="200" style="text-align: center;">
        <img src="./assets/logo_tw.png" style="width: 150px;margin-top: 30px;">
      </v-sheet>
      <v-divider thickness="3" class="border-opacity-100" color="grey-darken-4" vertical></v-divider>
      <v-col>
        <v-sheet color="#0F1112" class="h-screen">
          <Main />
        </v-sheet>
      </v-col>
      <v-divider thickness="3" class="border-opacity-100" color="grey-darken-4" vertical></v-divider>
      <v-sheet class="d-none d-md-block" color="#0F1112" width="200" style="text-align: center;">
      </v-sheet>
    </v-row>
  </div>
</template>

<script lang="ts" setup>
import Main from "./components/Main.vue"
</script>

<style>

</style>
