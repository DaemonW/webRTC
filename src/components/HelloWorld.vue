<script setup>
import {ref} from 'vue'

defineProps({
  msg: String
})

const count = ref(0)
</script>

<script>
export default {
  data() {
    return {
      width: 320,
      height: 0,
      streaming: false,
      video: null,
      canvas: null,
      photo: null,
      btn_cap: null

    }
  },
  mounted() {
    window.addEventListener('load', this.startup, false);
  },
  methods: {
    startup() {
      this.video = document.getElementById('video');
      this.canvas = document.getElementById('canvas');
      this.photo = document.getElementById('photo');
      this.btn_cap = document.getElementById('btn_cap');

      navigator.mediaDevices.getUserMedia({video: true, audio: false})
          .then(function (stream) {
            this.video.srcObject = stream;
            this.video.play();
          })
          .catch(function (err) {
            console.log("An error occurred: " + err);
          });
      let self = this;
      this.video.addEventListener('canplay', function (ev) {
        if (!self.streaming) {
          this.height = this.video.videoHeight / (this.video.videoWidth /this.width);

          // Firefox currently has a bug where the height can't be read from
          // the video, so we will make assumptions if this happens.

          if (isNaN(this.height)) {
            this.height = this.width / (4 / 3);
          }

          this.video.setAttribute('width', this.width);
          this.video.setAttribute('height', this.height);
          this.canvas.setAttribute('width', this.width);
          this.canvas.setAttribute('height', this.height);
          this.streaming = true;
        }
      }, false);
      this.btn_cap.addEventListener('click', function (ev) {
        console.log("Take Photo!")
        self.takePicture();
        ev.preventDefault();
      }, false);

      this.clearPhoto();
    },


    // Fill the photo with an indication that none has been
    // captured.

    clearPhoto() {
      var context = this.canvas.getContext('2d');
      context.fillStyle = "#AAA";
      context.fillRect(0, 0, this.canvas.width, this.canvas.height);

      var data = this.canvas.toDataURL('image/png');
      this.photo.setAttribute('src', data);
    },


    // Capture a photo by fetching the current contents of the video
    // and drawing it into a canvas, then converting that to a PNG
    // format data URL. By drawing it on an offscreen canvas and then
    // drawing that to the screen, we can change its size and/or apply
    // other changes before drawing it.

    takePicture() {
      var context = this.canvas.getContext('2d');
      if (this.width && this.height) {
        this.canvas.width = this.width;
        this.canvas.height = this.height;
        context.drawImage(this.video, 0, 0, this.width, this.height);

        var data = this.canvas.toDataURL('image/png');
        this.photo.setAttribute('src', data);
      } else {
        this.clearPhoto();
      }
    },
  }
}
</script>

<template>
  <h1>{{ msg }}</h1>
  <button type="button" @click="count++">count is: {{ count }}</button>
  <p></p>
  <div class="contentarea">
    <h1>
      MDN - WebRTC: Still photo capture demo
    </h1>
    <p>
      This example demonstrates how to set up a media stream using your built-in webcam, fetch an image from that
      stream, and create a PNG using that image.
    </p>
    <div class="camera">
      <video id="video">Video stream not available.</video>
      <button id="btn_cap">Take photo</button>
    </div>
    <canvas id="canvas">
    </canvas>
    <div class="output">
      <img id="photo" alt="The screen capture will appear in this box.">
    </div>
    <p>
      Visit our article <a href="https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Taking_still_photos">
      Taking still photos with WebRTC</a> to learn more about the technologies used here.
    </p>
  </div>
</template>

<style scoped>
a {
  color: #42b983;
}
</style>
