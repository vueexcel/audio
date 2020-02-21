<template>
  <div>
      <!-- Get json file -->
      <b-row class="mt-5" align-h="center">
      <b-col cols="4">
        <b-form-file v-model="audioFile" @change="getJsonFile"></b-form-file>
      </b-col>
    </b-row>
    <!-- show audio signals -->
    <b-row align-h="center">
      <b-col cols="3" align-self="center">
        <div ref="waveform" class="mt-5"></div>
        <div v-if="timestamp.length">
            <b-icon v-if="play" icon="pause-fill" variant="primary" class="cursor" style="width: 50px; height: 50px;" @click="playPauseFunction"></b-icon>
            <b-icon v-else icon="play-fill" variant="success" class="cursor" style="width: 50px; height: 50px;" @click="playPauseFunction"></b-icon>
        </div>
      </b-col>
    </b-row>
    <!-- show audio text -->
    <b-row align-h="center" v-if="wavesurfer && !play && currentTime === null">
      <b-col cols="8" align-self="center">
        <h4 class="text-info">Please click play button to play the audio</h4>
      </b-col>
    </b-row>
    <b-row align-h="center">
      <b-col cols="8" align-self="center">
        <p class="text-break">
          <span v-for="(time, index) in timestamp" :key="index" class="cursor" @dblclick="deleteWord(index)">
             <span 
              :class="{ 'text-danger': time.start_time <= currentTime && time.end_time >= currentTime,
                'font-weight-bolder': time.start_time <= currentTime && time.end_time >= currentTime }"
               v-if="!time.strikethrough">
              {{time.text}}
             </span>
            <del v-else>{{time.text}}</del>
          </span>
        </p>
      </b-col>
    </b-row>
  </div>
</template>

<script>
import WaveSurfer from 'wavesurfer.js'
export default {
  name: 'Audio',
  data () {
    return {
        play: false,
        url: '',
        jsonUrl: '',
        wavesurfer: null,
        currentTime: null,
        audioFile: null,
        timestamp: []
      }
    },
    methods: {
      // function to play or pause the audio 
      playPauseFunction() {
        this.play = !this.play
        this.wavesurfer.playPause()
      },
      // function to initialize the waveSurfer
      initialize () {
        this.wavesurfer = WaveSurfer.create({
            container: this.$refs.waveform,
            waveColor: 'violet',
            progressColor: 'red',
            barHeight: 2
        });
        this.wavesurfer.load(this.url)
        this.wavesurfer.on('audioprocess',() => {
            this.currentTime = this.wavesurfer.getCurrentTime()
            this.wavesurfer.params.container.style.opacity = 0.9;
        });
      },
      getJsonFile (event) {
        this.wavesurfer = null
        var reader = new FileReader();
        reader.onload = this.onReaderLoad;
        reader.readAsText(event.target.files[0])
      },
      onReaderLoad(event){
        var obj = JSON.parse(event.target.result);
        obj.sections.forEach(async section => {
        await this.getString(section.words)
        });
        this.url = obj.meta.url
        this.initialize()
      },
      getString (arrayForString) {
        for (let i = 0; i < arrayForString.length; i++) {
            this.timestamp.push(arrayForString[i])
        }
      },
      deleteWord (index) {
        this.timestamp[index].strikethrough = true
      }
    }
}
</script>

<style>
.cursor {
  cursor: pointer;
}
</style>