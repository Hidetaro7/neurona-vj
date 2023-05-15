<script setup>
// https://stackoverflow.com/questions/64248887/nodejs-converting-mtc-midi-timecode-quater-message-into-a-full-timecode-st

import { ref, computed, watch, onMounted } from "vue";
import * as JZZ from "jzz";

const slave = ref(JZZ.SMPTE()); // slave clock
const message = ref(null);
const ch = ref(null);
const isNoteOn = ref(false);
const isNoteOff = ref(false);
const note = ref(null);
const videoId = ref(null);
const videoPath = [
  "/20230521/01_20230521_op-Strange_Day.m4v",
  "/20230521/02_20230521_UTSUYUME.m4v",
  "/20230521/03_20230521_Z.A.C.O.m4v",
  "/20230521/04_20230521_lastboss.m4v",
  "/20230521/05_20230521_ed_1.m4v",
  "/20230521/06_20230521_ed_2.m4v",
  "/20230521/20230521_end_roll.m4v",
];
const videoEl = ref(null);

const port = JZZ().openMidiIn(/IAC/);
port
  .connect(function (msg) {
    slave.value.read(msg);
    console.log(msg.toString());
  })
  .connect(
    JZZ.Widget({
      _receive: (msg) => {
        message.value = msg;
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
  console.log(a[0], b[0]);
  if (!isNoteOn.value && a[0] === b[0]) return;
  //console.log(ch.value, isNoteOn.value);
  playVideo(a[0]);
  /* if (a[0] === 11) {
    playVideo(0);
  } else if (a[0] === 12) {
    playVideo(1);
  } else if (a[0] === 13) {
    playVideo(2);
  } else if (a[0] === 14) {
    playVideo(3);
  } else if (a[0] === 15) {
    playVideo(4);
  } */
});

const playVideo = (num) => {
  videoId.value = num;
  videoEl.value.addEventListener("canplay", () => {
    videoEl.value.play();
  });
};

onMounted(() => {
  videoEl.value.volume = 0.3;
  /* videoEl.value.addEventListener("ended", () => {
    videoEl.value.currentTime = 0;
  }); */
});

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
