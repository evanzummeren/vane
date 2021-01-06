<template>
  <div class="illustration__container">
    <div class="item" ref="item"></div>
    <h1 ref="title">{{title}}</h1>
    <div class="scroll">
      <img class="down" src="../assets/down.svg" />
    </div>
    <div class="circle" ref="circle"></div>
    <div class="circleBlack" ref="circleBlack"></div>

    <div class="progress__container">
      <svg class="progress__circle" height="100" width="100">
        <circle cx="50" cy="50" r="40" stroke="#000000" stroke-width="5" fill="#FFFFFF" />
      </svg> 
    </div>
  </div>
</template>

<script>
import * as PIXI from 'pixi.js'
import { gsap } from 'gsap'
import { PixiPlugin } from "gsap/PixiPlugin";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import axios from 'axios';

import image from "../assets/image.png"

window.PIXI = PIXI;
gsap.registerPlugin(PixiPlugin, ScrollTrigger);

export default {
  name: 'Illustration',
  data: function() {
    return {
      // illustration_id: 0,
      image: image,
      pixiObj: {},
      drawing: {},
      dispFilter: {},
      title: "",
      mobile: false,
      settings: {
        image: "../image.png",
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
  beforeMount() {
    // https://medium.com/@gtalarico/using-airtable-as-a-content-backend-e373cd0d9974
    // https://api.airtable.com/v0/app9eqtMYrYxXYmE8/Table%201?api_key=keyg694BdwuEIov7z

    let airtableResponse = {};
    let _vue = this;

    axios.get('https://api.airtable.com/v0/app9eqtMYrYxXYmE8/Table%201?api_key=keyg694BdwuEIov7z')
    .then(function (response) {
      // console.log(response.data.records);

      response.data.records.map((item) => {
        airtableResponse[item.fields.id] = {};
        if ("id" in item.fields) { airtableResponse[item.fields.id].id = item.fields.id }
        if ("title" in item.fields) { airtableResponse[item.fields.id].title = item.fields.title }
        if ("image" in item.fields) { airtableResponse[item.fields.id].image = item.fields.image[0].url } 
      })

      _vue.title = airtableResponse[_vue.$route.params.id].title;
    })
    .catch(function (error) {
      // handle error
      console.log(error);
    })

    // console.log(airtableResponse);
  },
  mounted() {
    window.scrollTo(0, 0);

    if (document.documentElement.clientWidth < 650) {
      this.mobile = true;
    }
    
    this.pixiObj = new PIXI.Application({ 
      width: window.innerWidth,
      height: window.innerHeight,
      transparent: true 
    });

    document.body.appendChild(this.pixiObj.view);

    this.showIntroduction()

    setTimeout(() => this.addDrawing(), 3200);
  },
  methods: {
    showIntroduction() { // Shows title animation
      this.$forceUpdate();
      let tl = gsap.timeline({
        delay: 0.3
      })

      tl.from(this.$refs.title, { 
        y: 10,
        duration: 2,
        opacity: 0
      }).to(this.$refs.title, {
        delay: 1,
        duration: 1,
        opacity: 0
      })
    },
    addDrawing() { // Adds drawing to canvas
      gsap.set(this.$refs.circle, {
        opacity: 1
      })

      this.drawing = PIXI.Sprite.from(this.image);

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
      const displacementSprite = PIXI.Sprite.from( require("../assets/displacement.jpg"));

      // Make sure the sprite is wrapping.
      displacementSprite.texture.baseTexture.wrapMode = PIXI.WRAP_MODES.REPEAT;

      this.dispFilter = new PIXI.filters.DisplacementFilter(displacementSprite);
      this.dispFilter.padding = 10;

      displacementSprite.position = art.position;

      this.pixiObj.stage.addChild(displacementSprite);
      art.filters = [this.dispFilter];

      this.dispFilter.scale.x = 300;
      this.dispFilter.scale.y = 600;


      let tl = gsap.timeline({
        delay: 0.3
      })

      tl.to(this.dispFilter, 2.5, {pixi:{scaleX: 0, scaleY: 0}})
        .to(this.dispFilter, 1.5, {pixi:{scaleX: 50, scaleY: 20}})
        .to(this.dispFilter, 1, {pixi:{scaleX: 0, scaleY: 0}})


      this.pixiObj.ticker.add(() => {
          // Offset the sprite position to make vFilterCoord update to larger value. Repeat wrapping makes sure there's still pixels on the coordinates.
          displacementSprite.x++;
          // Reset x to 0 when it's over width to keep values from going to very huge numbers.
          if (displacementSprite.x > displacementSprite.width) { displacementSprite.x = 0; }
      });
    },
    scrollAnim() {
      let tl = gsap.timeline({
        repeat: 2,
        repeatDelay: .5
      })

      tl.to(".down", {
        duration: 1.5,
        y: 20,
        opacity: 0
      })
    },
    rotateByScroll() {
      window.scrollTo(0, 0);

      gsap.to(".down", {
        opacity: 1,
        delay: 1,
        onComplete: this.scrollAnim
      })

      let art = this.drawing;
      art.scale.set(this.mobile ? this.settings.mobile.scale : this.settings.desktop.scale);
      art.anchor.set(0.5);

      let scrollTriggerSettings = {
        trigger: ".item",
        scrub: 2,
        start: 100,
        end: parseInt(window.getComputedStyle(document.querySelector(".item")).height, 10) - window.innerHeight - 400        
      }

      gsap.set(".progress__container", {
        opacity: 1
      })

      gsap.to(".progress__circle", {
        scrollTrigger: scrollTriggerSettings,
        strokeDashoffset:0
      })

      gsap.to(art, { 
        pixi: { 
          scale: this.mobile ? this.settings.mobile.transformedScale : this.settings.desktop.transformedScale, 
          rotation: -360
        },
        scrollTrigger: scrollTriggerSettings,
        onComplete: this.endOfScroll
      });
    },
    endOfScroll() {
      let art = this.drawing,
         _vue = this;

      gsap.to(".progress__container", 1, {
        opacity: 0, ease: "back.inOut(1)"
      })
         
      gsap.set(_vue.$refs.circleBlack, {
        opacity: 1
      })

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

      // Push router
      setTimeout(() => {
        window.scrollTo(0, 0);
        // needs more elegant fix
        // fix https://stackoverflow.com/questions/32106155/can-you-force-vue-js-to-reload-re-render
        this.$router.push(({ path: `../../illustration/2` }));
        location.reload();
      }, 4000);
      
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.progress__container {
  position: fixed;
  bottom: 1rem;
  left: 1rem;
  opacity: 0;
}

.progress__circle {
  transform: rotate(-90deg);
  stroke-dasharray: 251; /* (2PI * 40px) */
  stroke-dashoffset: 251;
  /* animation: offsettozero 5s linear forwards; */
}

/* @keyframes offsettozero {
  to {
    stroke-dashoffset: 0;
  }
} */

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
  opacity: 0;
}

.circleBlack {
  border-radius: 100%;
  background: black;
  width: 10px;
  height: 10px;
  position: fixed;
  left: 50%;
  top: 50%;
  opacity: 0;
  transform: translate(-50%,-50%);
}

.illustration__container {
  text-align: center;
  width: 100vw;
}

h1 {
  color: white;
  font-family: 'shar-medium';
  position: fixed;
  font-weight: 100;
  line-height: 100vh;
  width: 100vw;
  top: 0;
  left: 0;
  margin: 0;
  z-index: 100;
}

.down {
  position: fixed;
  left: 47%;
  bottom: 3rem;
  width: 25px;
  z-index: 100;
  opacity: 0;
}
</style>
