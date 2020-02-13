<template>
  <div id="app">
    <Video3d
      :key="num"
      :video="video"
      :depthMap="depth"
      :mouse="isMouse"
      :mouse-sensitivity="mouseSensitivity"
      :automatedPositions="animateXY"
      :animationSpeed="animationSpeed"
      @controlLock="controlLock"
    />
    <div id="buttonContainer">
      <div>
        <span>Backgrounds:</span>
        <button
          @click="setBackground('mountain')"
          :class="{'btn--active': currentBackground === 'mountain'}"
        >Mountain</button>
        <button
          @click="setBackground('game')"
          :class="{'btn--active': currentBackground === 'game'}"
        >Game</button>
        <button
          @click="setBackground('air')"
          :class="{'btn--active': currentBackground === 'air'}"
        >Air Balloon</button>
        <button
          @click="setBackground('yoga')"
          :class="{'btn--active': currentBackground === 'yoga'}"
        >Yoga</button>
      </div>
      <div>
        <span>Click image to play video</span>
      </div>
      <div>
        <span>Interactions:</span>
        <button
          @click="updateMouse(false)"
          :class="{'btn--active': !isMouse, 'btn--locked': lockControls}"
        >Animate X&Y</button>
        <button
          @click="updateMouse(true)"
          :class="{'btn--active': isMouse, 'btn--locked': lockControls}"
        >Mouse Move</button>
      </div>
    </div>
  </div>
</template>

<script>
import Video3d from "./components/Video3d";
import mountain from "@/assets/video.mp4";
import mountainDepth from "@/assets/video-depth.jpg";
import game from "@/assets/gameBIG.mp4";
import gameDepth from "@/assets/game-depth.jpg";
import airballoon from "@/assets/airballoon.mp4";
import airballoonDepth from "@/assets/airballoon-depth.jpg";
import yoga from "@/assets/yoga.mp4";
import yogaDepth from "@/assets/yoga-depth.jpg";
export default {
  name: "App",
  components: {
    Video3d
  },
  data() {
    return {
      video: mountain,
      depth: mountainDepth,
      isMouse: true,
      mouseSensitivity: 40,
      animateXY: [
        {
          x: 0,
          y: 1
        },
        {
          x: 1,
          y: 0
        },
        {
          x: 1,
          y: 1
        },
        {
          x: 0,
          y: 0
        }
      ],
      animationSpeed: 2000,
      lockControls: false,
      num: 0,
      currentBackground: "mountain"
    };
  },
  methods: {
    controlLock(bool) {
      this.lockControls = bool;
    },
    setBackground(val) {
      this.currentBackground = val;
      this.controlLock(false);
      if (val === "mountain") {
        this.video = mountain;
        this.depth = mountainDepth;
        this.animateXY = [
          {
            x: 0,
            y: 1
          },
          {
            x: 1,
            y: 0
          },
          {
            x: 1,
            y: 1
          },
          {
            x: 0.01,
            y: 0.01
          }
        ];
        this.animationSpeed = 2000;
        this.num++;
      }
      if (val === "game") {
        this.video = game;
        this.depth = gameDepth;
        this.mouseSensitivity = 100;
        this.animateXY = [
          {
            x: 0.49,
            y: 0.09
          },
          {
            x: 0.8,
            y: 0.52
          },
          {
            x: 0.51,
            y: 0.92
          },
          {
            x: 0.2,
            y: 0.5
          },
          {
            x: 0,
            y: 0
          }
        ];
        this.animationSpeed = 1500;
        this.num++;
      }
      if (val === "air") {
        this.video = airballoon;
        this.depth = airballoonDepth;
        this.mouseSensitivity = 100;
        this.num++;
      }
      if (val === "yoga") {
        this.video = yoga;
        this.depth = yogaDepth;
        this.mouseSensitivity = 100;
        this.num++;
      }
    },
    updateMouse(bool) {
      if (!this.lockControls) {
        this.isMouse = bool;
        this.num++;
      }
    }
  }
};
</script>

<style lang="scss">
html,
body {
  margin: 0;
  padding: 0;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 0px;
  padding: 0px;
}

#buttonContainer {
  display: flex;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  max-width: 100vw;
  justify-content: space-between;
  align-items: space-between;
  padding: 10px 0px;
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(5px);
  > div {
    padding: 0px 30px;
    span {
      color: #fff;
      text-transform: uppercase;
      margin: 0;
    }
  }
  button {
    padding: 5px 10px;
    display: inline-block;
    border-radius: 3px;
    border: none;
    background: #ccc;
    font-size: 12px;
    font-weight: bold;
    color: #333;
    margin-left: 4px;
    margin-right: 4px;
    cursor: pointer;
    &:hover {
      background: rgb(39, 212, 172);
    }
    &.btn--active {
      background: rgb(92, 255, 217);
      &:hover {
        background: rgb(39, 212, 172);
      }
    }
    &.btn--locked {
      cursor: not-allowed;
    }
  }
}
</style>
