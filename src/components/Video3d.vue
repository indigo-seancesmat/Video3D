<template>
  <div ref="video3d" class="video3d">
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script>
import * as PIXI from "pixi.js";
import debounce from "lodash/debounce";

export default {
  name: "Video3d",
  props: {
    depthMap: {
      type: String,
      required: true
    },
    video: {
      type: String,
      required: true
    },
    mouse: {
      type: Boolean,
      default: false
    },
    mouseSensitivity: {
      type: Number,
      default: 40
    },
    animationSpeed: {
      type: Number,
      default: 1000
    },
    automatedPositions: {
      type: Array,
      default: () => {
        return [
          {
            x: 0.53,
            y: 0.06
          },
          {
            x: 0.7,
            y: 0.18
          },
          {
            x: 0.94,
            y: 0.9
          },
          {
            x: 0.34,
            y: 0.9
          },
          {
            x: 0.32,
            y: 0.77
          },
          {
            x: 0.07,
            y: 0.08
          },
          {
            x: 0,
            y: 0
          }
        ];
      }
    }
  },
  data() {
    return {
      app: null,
      x: 0,
      y: 0,
      width: window.innerWidth,
      height: window.innerHeight
    };
  },
  beforeDestroy() {
      console.log('destroy')
  },
  mounted() {
    this.width = window.innerWidth;
    this.height = window.innerHeight;

    this.renderer = new PIXI.Renderer({
      view: this.$refs.canvas,
      width: this.width,
      height: this.height,
      resolution: window.devicePixelRatio,
      autoDensity: true
    });
    this.stage = new PIXI.Container();

    this.videoTexture = PIXI.Texture.from(this.video);
    this.nativeVideo = this.videoTexture.baseTexture.resource;
    this.nativeVideo.autoPlay = false;
    this.nativeVideo.preload = "auto";

    this.videoSprite = new PIXI.Sprite(this.videoTexture);
    this.videoSprite.texture.baseTexture.on("loaded", () => {
      this.setScale();
    });
    this.nativeVideo.source.onended = () => {
      this.nativeVideo.source.currentTime = 0;
      this.$emit("controlLock", false);
      this.MouseStuff();
    };
    this.videoSprite.anchor.x = 0.5;
    this.videoSprite.anchor.y = 0.5;
    this.stage.addChild(this.videoSprite);

    this.depthMapCont = new PIXI.Sprite.from(this.depthMap);
    this.depthMapCont.anchor.x = 0.5;
    this.depthMapCont.anchor.y = 0.5;
    this.stage.addChild(this.depthMapCont);

    window.addEventListener("resize", debounce(this.handleResize, 100));
    this.$refs.canvas.onclick = e => {
      this.play();
      this.$emit("controlLock", true);
      console.log(`x: ${e.clientX / this.width} y: ${e.clientY / this.height}`);
    };
    this.MouseStuff();
  },
  methods: {
    play() {
      window.removeEventListener("mousemove", this.mouseMove);
      this.x = this.displacementFilter.scale.x;
      this.y = this.displacementFilter.scale.y;
      this.ticker = new PIXI.Ticker();
      this.ticker.add(() => {
        this.displacementFilter.scale.x = this.x;
        this.displacementFilter.scale.y = this.y;
        this.renderer.render(this.stage);
      });
      this.ticker.start();
      this.$anime({
        targets: this.$data,
        easing: "linear",
        duration: 500,
        x: 0,
        y: 0,
        complete: () => {
          this.stage.filters = [];
          this.videoTexture.baseTexture.resource.source.play();
          this.ticker.stop();
        }
      });
    },
    handleResize() {
      this.width = window.innerWidth;
      this.height = window.innerHeight;
      this.renderer.resize(this.width, this.height);
      this.setScale();
    },
    setScale() {
      this.containerRatio = this.width / this.height;
      this.videoRatio = this.videoTexture.width / this.videoTexture.height;
      if (this.containerRatio > this.videoRatio) {
        this.scale = this.width / this.videoTexture.width;
      } else {
        this.scale = this.height / this.videoTexture.height;
      }
      this.setWidthHeightPos();
    },
    setWidthHeightPos() {
      this.videoSprite.x = this.renderer.screen.width / 2;
      this.videoSprite.y = this.renderer.screen.height / 2;
      this.videoSprite.scale.x = this.scale;
      this.videoSprite.scale.y = this.scale;
      this.depthMapCont.position.x = this.renderer.screen.width / 2;
      this.depthMapCont.position.y = this.renderer.screen.height / 2;
      this.depthMapCont.scale.x = this.scale;
      this.depthMapCont.scale.y = this.scale;
    },
    getXPos(percent) {
      // percentage between 0-1
      return (this.width / 2 - this.width * percent) / 40;
    },
    getYPos(percent) {
      // percentage between 0-1
      return (this.height / 2 - this.height * percent) / 40;
    },
    mouseMove(e) {
      this.displacementFilter.scale.x =
        (this.width / 2 - e.clientX) / this.mouseSensitivity;
      this.displacementFilter.scale.y =
        (this.height / 2 - e.clientY) / this.mouseSensitivity;
    },
    MouseStuff() {
      if (this.mouse === true) {
        this.displacementFilter = new PIXI.filters.DisplacementFilter(
          this.depthMapCont
        );
        this.stage.filters = [this.displacementFilter];
        window.addEventListener("mousemove", this.mouseMove);

        this.ticker = new PIXI.Ticker();
        this.ticker.add(() => {
          this.renderer.render(this.stage);
        });
        this.setScale();
        this.ticker.start();
      } else {
        this.displacementFilter = new PIXI.filters.DisplacementFilter(
          this.depthMapCont
        );
        this.stage.filters = [this.displacementFilter];
        this.ticker = new PIXI.Ticker();
        this.ticker.add(() => {
          this.displacementFilter.scale.x = this.x;
          this.displacementFilter.scale.y = this.y;
          // console.log("displacementFilter");
          this.renderer.render(this.stage);
        });
        this.setScale();
        this.ticker.start();
        this.tl = this.$anime.timeline({
          easing: "linear",
          duration: this.animationSpeed,
          loop: true
        });

        this.automatedPositions.forEach(cords => {
          if (cords.x === 0 && cords.y === 0) {
            this.tl.add({
              targets: this.$data,
              x: 0,
              y: 0
            });
          } else {
            this.tl.add({
              targets: this.$data,
              x: this.getXPos(cords.x),
              y: this.getXPos(cords.y)
            });
          }
        });
      }
    }
  }
};
</script>

<style lang="scss" scoped>
.video3d {
  height: 100vh;
  width: 100vw;

  h1 {
    color: blue;
  }
}
</style>
