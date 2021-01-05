<template>
  <div id="app">
    <div class="item" ref="item"></div>
    <span>Scroll down to rotate</span>
    <div class="circle" ref="circle"></div>
    <div class="circleBlack" ref="circleBlack"></div>

    <!-- <div class="item" ref="item">a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>b<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/>a<br/></div> -->
    <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
  </div>
</template>

<script>
import * as PIXI from 'pixi.js'
import { gsap } from 'gsap'
import { PixiPlugin } from "gsap/PixiPlugin";
import { ScrollTrigger } from "gsap/ScrollTrigger";

window.PIXI = PIXI;

gsap.registerPlugin(PixiPlugin, ScrollTrigger);

export default {
  name: 'App',
  data: function() {
    return {
      pixiObj: {},
      drawing: {},
      dispFilter: {},
      mobile: false,
      settings: {
        image: "./image.png",
        desktop: {
          scale: .3,
          transformedScale: .35
        },
        mobile: {
          scale: .2,
          transformedScale: .25
        }


        // image: "./image.png",
        // desktop: {
        //   scale: .3,
        //   transformedScale: .35
        // },
        // mobile: {
        //   scale: .2,
        //   transformedScale: .25
        // }

      }
    }
  },
  methods: {
    addDrawing() {
      // this.drawing = PIXI.Sprite.from( require("./assets/image.png"));
      this.drawing = PIXI.Sprite.from(this.settings.image);

      let art = this.drawing;

      art.scale.set(this.mobile ? this.settings.mobile.scale : this.settings.desktop.scale);
      art.anchor.set(0.5);

      art.x = this.pixiObj.screen.width / 2;
      art.y = this.pixiObj.screen.height / 2;

      gsap.to(this.$refs.circle, 2.5, {
        scale: 200, rotation: 0, ease: "back.inOut(1)"
      })

      gsap.from(art, 5, {
        pixi: { 
          scale: 0, 
          rotation:160
        },
        onComplete: this.rotateByScroll
      });

      this.pixiObj.stage.addChild(art);

      const displacementSprite = PIXI.Sprite.from( require("./assets/displacement.jpg"));

      // Make sure the sprite is wrapping.
      displacementSprite.texture.baseTexture.wrapMode = PIXI.WRAP_MODES.REPEAT;
      this.dispFilter = new PIXI.filters.DisplacementFilter(displacementSprite);
      this.dispFilter.padding = 10;

      displacementSprite.position = art.position;

      this.pixiObj.stage.addChild(displacementSprite);

      art.filters = [this.dispFilter];

      this.dispFilter.scale.x = 300;
      this.dispFilter.scale.y = 600;

      gsap.to(this.dispFilter, 3, {pixi:{scaleX: 0, scaleY: 0}});


      this.pixiObj.ticker.add(() => {
          // Offset the sprite position to make vFilterCoord update to larger value. Repeat wrapping makes sure there's still pixels on the coordinates.
          displacementSprite.x++;
          // Reset x to 0 when it's over width to keep values from going to very huge numbers.
          if (displacementSprite.x > displacementSprite.width) { displacementSprite.x = 0; }
      });


    },
    rotateByScroll() {
      let art = this.drawing;

      art.scale.set(this.mobile ? this.settings.mobile.scale : this.settings.desktop.scale);
      art.anchor.set(0.5);



      gsap.to(art, 
        { pixi: { 
            scale: this.mobile ? this.settings.mobile.transformedScale : this.settings.desktop.transformedScale, 
            rotation: -360
          },
          scrollTrigger: {
            trigger: ".item",
            scrub: 2,
            start: 100,
            end: parseInt(window.getComputedStyle(document.querySelector(".item")).height, 10) - window.innerHeight - 400
          },
          onComplete: this.endOfScroll
        });
    },
    endOfScroll() {
      let art = this.drawing;
          let _vue = this;

      gsap.to(_vue.$refs.circleBlack, 3, {
        scale: 200, rotation: 0, ease: "back.inOut(1)", delay: 1
      })

      gsap.to(this.dispFilter, 7, {delay: "=-3", pixi:{scaleX: 300, scaleY: 600}});

      gsap.to(art, 7, {
        pixi:{scale: 100, rotation: 10},
        ease: "power3.inOut",
        delay: 1, 
        onStart: function() {
          console.log('Start of transition')
        }
      });
      
    }
  },
  mounted: function() {
    if (document.documentElement.clientWidth < 650) {
      this.mobile = true;
    }

    this.pixiObj = new PIXI.Application({ 
      width: window.innerWidth,
      height: window.innerHeight,
      transparent: true 
    });

    document.body.appendChild(this.pixiObj.view);

    this.addDrawing();
  }
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
  overflow-x: hidden;
  background: black;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

canvas {
  position: fixed;
  pointer-events: none;
  top: 0;
}

.item {
  height: 400vh;
}

.circle {
  border-radius: 100%;
  background: white;
  width: 10px;
  height: 10px;
  position: fixed;
  left: 50%;
  top: 50%;
  transform: translate(-50%,-50%);
}

.circleBlack {
  border-radius: 100%;
  background: black;
  width: 10px;
  height: 10px;
  position: fixed;
  left: 50%;
  top: 50%;
  transform: translate(-50%,-50%);
}
</style>
