<script setup>
import { ref, watch, onMounted } from "vue";
import { WebMidi } from "webmidi";
const devices = ref([]);
const note = ref();
const videoId = ref(null);

/* TODO: 5/21 イーストコート */
const videoPath = [
  "/20230521/01_20230521_op-Strange_Day.m4v",
  "/20230521/02_20230521_UTSUYUME.m4v",
  "/20230521/03_20230521_Z.A.C.O.m4v",
  "/20230521/04_ZACO_movie.m4v",
  "/20230521/05_20230521_lastboss.m4v",
  "/20230521/06_20230521_ed.m4v",
  "/20230521/07_20230521_shimobe.m4v",
  "/20230521/08_20230521_end_roll.m4v",
];

/* TODO: 5/23 キャッツホール */
/* const videoPath = [
  "/20230523/01_20230523_op-StrangeDay.m4v",
  "/20230521/03_20230521_Z.A.C.O.m4v",
  "/20230521/04_ZACO_movie.m4v",
  "/20230523/03_20230523_soranihabureba.m4v",
  "/20230523/04_20230523_ed.m4v",
  "/20230521/07_20230521_shimobe.m4v",
  "/20230523/05_20230523 end roll.m4v",
]; */
const videoEl = ref(null);
// Enable WEBMIDI.js and trigger the onEnabled() function when ready

WebMidi.enable()
  .then(onEnabled)
  .catch((err) => console.log(err));

// Function triggered when WEBMIDI.js is ready
function onEnabled() {
  // Display available MIDI input devices
  if (WebMidi.inputs.length < 1) {
    devices.value.push("No device detected.");
  } else {
    WebMidi.inputs.forEach((device, index) => {
      devices.value.push(`${index}: ${device.name}`);
    });
  }
  // const mySynth = WebMidi.inputs[0];
  const mySynth = WebMidi.getInputByName("IAC driver UAD Console MIDI");

  mySynth.channels[16].addListener("noteon", (e) => {
    note.value = e.note.number;
  });
}

watch(note, (a, b) => {
  //if (a === b) return;
  playVideo(a);
});
const playVideo = (num) => {
  videoId.value = num;
  videoEl.value.addEventListener("canplay", () => {
    videoEl.value.play();
  });
};

onMounted(() => {
  videoEl.value.volume = 0.3;
});

// about video fullscreen
const videoFullScreen = () => {
  videoEl.value.requestFullscreen();
};
/* document.addEventListener(
  "keydown",
  (e) => {
    if (e.key === "Enter") {
      videoFullScreen();
    }
  },
  false
); */
</script>

<template>
  <div>
    {{ devices }}
    <p>note: {{ note }}</p>
    <video
      class="preview-video"
      :src="videoPath[videoId]"
      ref="videoEl"
    ></video>
    <p class="button fullscreen">
      <button @click="videoFullScreen()">Full screen</button>
    </p>
  </div>
</template>

<style scoped>
.preview-video {
  width: 100%;
  aspect-ratio: auto;
}

.button.fullscreen button {
  margin: 40px 0;
  font-weight: bold;
  font-size: 2rem;
}
</style>
