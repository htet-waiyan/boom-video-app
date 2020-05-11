<template>
  <div class="container-fluid boom-container bg-dark">
    <!-- <div class="meeting_registraion">
      <input type="text" placeholder="username"/>
      <button></button>
    </div> -->
    <boom-video/>
    <!-- <router-view/> -->
  </div>
</template>

<script>
import BoomVideo from './components/BoomVideo.vue';

export default {
  components: {
    BoomVideo,
  },
};
</script>

<style>
html,body {
  width: 100%;
  height: 100%;
}
.boom-container {
  height: 100%;
  overflow: hidden;
}
</style>
