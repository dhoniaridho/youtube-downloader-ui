<script setup lang="ts">
import axios from 'axios';
import {reactive} from 'vue';
import Loading from './components/Loading.vue';

const state = reactive({
  url: '',
  type: 'mp3',
  blobUrl: '',
  isLoading: false,
  hasError: false,
  isSuccess: false,
  activeTabs: 'home',
  medata: {
    title: '',
    duration: '',
  },
});
async function HandleSubmit() {
  state.isLoading = true;
  try {
    let {data: response} = await axios.get(`https://dhoniaridho-yt-download.herokuapp.com/api/stream?url=${state.url}&type=${state.type}`, {
      responseType: 'blob',
    });
    let {data: medata} = await axios.get(`https://dhoniaridho-yt-download.herokuapp.com/api/stream/medata?url=${state.url}`);

    state.medata = medata;
    state.blobUrl = URL.createObjectURL(response);
    state.isSuccess = true;
    state.activeTabs = 'preview';
  } catch (e) {
    state.hasError = true;
    state.isSuccess = false;
  } finally {
    state.isLoading = false;
  }
}

function downloadFile() {
  const link = document.createElement('a');
  link.download = `${state.medata.title}.${state.type}`;
  link.href = state.blobUrl;
  link.click();
}
</script>

<template>
  <div class="h-screen w-full flex place-items-center justify-center bg-gradient-to-r from-indigo-500 via-purple-500 to-pink-500">
    <div class="bg-slate-400 border w-11/12 h-5/6 md:w-[40%] md:min-h-[60%] overflow-hidden rounded-lg bg-opacity-40 backdrop-blur-md backdrop-filter">
      <div class="w-full h-full flex justify-center flex-col text-center gap-6 py-8 px-5" v-if="!state.isLoading && state.activeTabs === 'home'">
        <h1 class="text-4xl font-extrabold tracking-tighter">Youtube Downloader</h1>
        <form class="flex flex-col gap-5" @submit.prevent="HandleSubmit">
          <input class="form-input" type="text" name="url" id="" v-model="state.url" />
          <select v-model="state.type" class="focus:outline-none rounded-lg py-2 px-3">
            <option value=""></option>
            <option value="mp3">Audio Mp3</option>
            <option value="mp4">Video</option>
          </select>
          <button class="button-primary bg-rose-500" type="submit">submit</button>
        </form>
      </div>
      <div class="w-full h-full flex flex-col" v-if="state.isSuccess && state.activeTabs === 'preview'">
        <div class="h-5 bg-white bg-opacity-30 w-full py-8 px-5 flex justify-between items-center">
          <button @click="state.activeTabs = 'home'">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" />
            </svg>
          </button>
        </div>
        <div class="flex flex-1 justify-center items-center flex-col gap-5 py-8 px-5">
          <h2 class="text-xl font-semibold text-center">{{ state.medata.title }}</h2>
          <audio v-if="state.type === 'mp3'" class="w-full" controls>
            <source :src="state.blobUrl" type="audio/mpeg" />
          </audio>
          <video class="max-h-[13rem]" v-if="state.type === 'mp4'" controls>
            <source :src="state.blobUrl" type="video/mp4" />
          </video>
          <button @click="downloadFile" class="button-primary">DOWNLOAD</button>
        </div>
      </div>
      <Loading v-if="state.isLoading" />
    </div>
  </div>
</template>

<style>
.form-input {
  @apply py-2 px-3 rounded-lg border-rose-400 border w-full focus:outline-none;
}
.button-primary {
  @apply bg-rose-600 uppercase px-3 py-2 rounded-lg shadow-lg shadow-rose-500 text-white w-full hover:brightness-110 hover:shadow-rose-500 hover:shadow-xl;
}
</style>
