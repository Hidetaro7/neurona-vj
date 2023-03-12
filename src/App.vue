<script setup>
// https://stackoverflow.com/questions/64248887/nodejs-converting-mtc-midi-timecode-quater-message-into-a-full-timecode-st

import { ref, computed, watch } from "vue";
import * as JZZ from "jzz";

var slave = ref(JZZ.SMPTE()); // slave clock
const message = ref(null);
const ch = ref(null);
const isNoteOn = ref(false);
const isNoteOff = ref(false);
const note = ref(null);
const videoId = ref(null);
const videoPath = [
  "/LIVE0319_op.m4v",
  "/20230319LIVE_ed.m4v",
  "/LIVE0319_endroll.m4v",
];
const videoEl = ref(null);

const port = JZZ().openMidiIn(0);
port
  .connect(function (msg) {
    slave.value.read(msg);
  })
  .connect(
    JZZ.Widget({
      _receive: (msg) => {
        // message.value = msg;
        ch.value = msg.getChannel(); // チャンネル番号は0から取得になる（1ch -> 0）
        isNoteOn.value = msg.isNoteOn();
        isNoteOff.value = msg.isNoteOff();
        note.value = isNoteOff.value ? null : msg.getNote();
      },
    })
  )
  .close();

const timeCodeString = computed(() => slave.value.toString());

watch([ch, isNoteOn], (a, b) => {
  if (a[0] === 13) {
    console.log(a);
    playVideo(0);
  } else if (a[0] === 14) {
    playVideo(1);
  } else if (a[0] === 15) {
    playVideo(2);
  }
});

const playVideo = (num) => {
  videoId.value = num;
  videoEl.value.addEventListener("canplay", () => {
    videoEl.value.play();
  });
};

// about video fullscreen
const videoFullScreen = () => {
  videoEl.value.requestFullscreen();
};
document.addEventListener(
  "keydown",
  (e) => {
    if (e.key === "Enter") {
      videoFullScreen();
    }
  },
  false
);
</script>

<template>
  <p class="timecode">
    {{ timeCodeString }}
  </p>
  <p>{{ message }}</p>
  <p>channel: {{ ch }}</p>
  <p>isNoteOn: {{ isNoteOn }}</p>
  <p>isNoteOff: {{ isNoteOff }}</p>
  <p>note: {{ note }}</p>
  <p><button @click="videoFullScreen(true)">フルスクリーン</button></p>
  <div>
    {{ videoId }}番のVideoの再生を開始
    <video
      class="preview-video"
      :src="videoPath[videoId]"
      ref="videoEl"
    ></video>
  </div>
</template>

<style scoped>
.timecode {
  font-size: 2rem;
}

.preview-video {
  width: 100%;
  aspect-ratio: auto;
}
</style>
