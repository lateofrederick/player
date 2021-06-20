<template>
  <div
    class="container"
    @drop="dropHandler($event)"
    @dragover="dragOverHandler($event)"
  >
    <div class="children">
      <audio
        @timeupdate="setPercentage"
        ref="audio"
        id="audio"
        @ended="displayControls"
        src=""
      >
      </audio>
      <div class="song drop_zone" :class="{ songDragged: songDragged }">
        <div v-if="!songDragged">
          <h3>Drag and drop a song here to play.</h3>
        </div>
        <div
          v-if="songDragged"
          class="image"
          :class="{ songDragged: songDragged }"
        >
          <img
            width="200px"
            height="200px"
            ref="image"
            :src="imageSrc"
            alt="song image"
          />
        </div>
      </div>
      <div class="progress-info">
      <div ref="progress" class="progress" @click="songProgressHandler($event)">
        <div ref="bar" class="bar"></div>
      </div>
      </div>
      <div id="controls">
        <button ref="play" :disabled="!songDragged" id="play" @click="playAudio">play</button>
        <button ref="pause" :hidden="!startedPlaying" id="pause" @click="pauseAudio">pause</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      file: "",
      imageSrc: "",
      songDragged: false,
      startedPlaying: false
    };
  },
  methods: {
    displayControls() {
      this.$refs.play.style.display = "block";
      this.$refs.pause.style.display = "none";
      this.$refs.bar.style.width = "0%";
      this.songDragged = false;
    },
    playAudio() {
      this.$refs.play.style.display = "none";
      this.$refs.pause.style.display = "block";
      if (!this.startedPlaying) {
        this.playFile();
      } else {
        this.$refs.audio.play();
      }
      
    },
    pauseAudio() {
      this.$refs.audio.pause();
      this.$refs.pause.style.display = "none";
      this.$refs.play.style.display = "block";
      this.startedPlaying = !this.startedPlaying;
    },
    setPercentage() {
      // function to set the percentage of the audio played
      this.$refs.bar.style.width =
        parseInt(
          (this.$refs.audio.currentTime / this.$refs.audio.duration) * 100,
          10
        ) + "%";
    },
    playFile() {
      let freader = new FileReader();

      freader.onload = (e) => {
        this.$refs.audio.src = e.target.result;
        this.$refs.audio.play();
      };

      freader.readAsDataURL(this.file);
      console.log(this.file);
    },
    async getSongDetails(file) {
      // this.file = e.target.files[0];
      if (!file) {
        alert("Please select the file to upload first!");
        return;
      }
      // console.log(this.file);
      let base64 = this.convertImagetoBase64(file);
      this.imageSrc = base64;
    },
    async convertImagetoBase64(file) {
      var jsmediatags = require("jsmediatags");
      let base64 = "heyyy";
      await jsmediatags.read(file, {
        onSuccess: (tag) => {
          let image = tag.tags.picture;

          if (image) {
            let base64String = "";
            for (let i = 0; i < image.data.length; i++) {
              base64String += String.fromCharCode(image.data[i]);
            }
            base64 =
              "data:" + image.format + ";base64," + window.btoa(base64String);
            // this.$refs.image.$el.setAttribute("src", base64);
            this.imageSrc = base64;
            return base64;
          } else {
            base64 = "couldn't get image";
          }
        },
        onError: function (error) {
          console.log(":(", error.type, error.info);
        },
      });
      return base64;
    },
    dropHandler(ev) {
      ev.preventDefault();

      if (ev.dataTransfer.items) {
        this.songDragged = true;
        this.file = ev.dataTransfer.items[0].getAsFile();
        this.getSongDetails(this.file);
      } else {
        // using data transfer interface
        for (let i = 0; i < ev.dataTransfer.files.length; i++) {
          const file = ev.dataTransfer.files[i];
          console.log("... file[" + i + "].name = " + file.name);
        }
        this.songDragged = true;
      }
    },
    dragOverHandler(event) {
      event.preventDefault();
    },
    songProgressHandler(event) {
      const width = this.$refs.progress.clientWidth;
      const clickX = event.offsetX;
      const duration = this.$refs.audio.duration;
      this.$refs.audio.currentTime = (clickX / width) * duration;
      console.log(clickX);
      console.log(width);
    }
  },
  mounted() {
    if (this.$refs.audio.paused) {
      this.displayControls();
    }
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.container {
  position: relative;
  height: 80vh;
}
img {
  object-fit: fill;
}
.children {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

#controls {
  margin: 10px;
  padding: 10px;
  position: relative;
  width: 100%;
  display: flex;
  justify-content: center;
}

#play, #pause {
  transform: translate(-50%, -50%);
}

.progress {
  border-bottom: 1px solid black;
  width: 200px;
  margin-left: 16px;
}

.progress-info {
  height: 5px;
  margin: 10px;
  padding: 10px;
}

.bar {
  height: 2px;
  background-color: green;
  width: 0;
}
.song {
  height: 200px;
  width: 200px;
  padding: 5px;
  margin-left: 30px;
}

h3 {
  /* text-align: center; */
  vertical-align: middle;
}

.songDragged {
  border: none;
}
</style>
