<template>
  <div>
    <button @click="startPlay" class="lever" id="lever" :class="{'running': running}"></button>
  </div>
</template>

<script>
  import * as PIXI from 'pixi.js'
  import gsap from "gsap";

  // or all tools are exported from the "all" file (excluding bonus plugins):
  import { Draggable } from "gsap/Draggable";

  // don't forget to register plugins
  gsap.registerPlugin(Draggable);

  export default {
    name: 'Slots',
    data(){
      return {
        reels: [],
        running: false,
        tweening: [],
        width: 800,
        height: 500,
        sprites: {}
      }
    },
    props: {
    },
    mounted(){
        const _self = this;

        const app = new PIXI.Application({
          backgroundColor: 0xefefef,
          width: _self.width,
          height: _self.height
        });

        document.body.appendChild(app.view);

        const REEL_WIDTH = 216;
        const SYMBOL_SIZE = 130;
        const margin = (app.screen.height - SYMBOL_SIZE * 3);

        const style = new PIXI.TextStyle({
          fontFamily: 'Arial',
          fontSize: 75,
          fontWeight: 'bold',
          fill: ['#FFD700', '#ffffff', '#FFD700'], // gradient
          wordWrap: true,
          wordWrapWidth: 440,
        });

        // Load sprites
        app.loader
          .add('one', '/assets/images/seven-black.PNG')
          .add('two', '/assets/images/triple7-yellow.PNG')
          .add('three', '/assets/images/seven-text_transparent.PNG')
          .add('four', '/assets/images/seven-yellow.PNG')
          .add('five', '/assets/images/triple7-black.PNG')
          .load(onAssetsLoaded);

        // onAssetsLoaded handler builds the example.
        function onAssetsLoaded() {

          // Create different slot symbols.
          const slotTextures = [
            PIXI.Texture.from('/assets/images/seven-text_transparent.PNG'),
            PIXI.Texture.from('/assets/images/triple7-yellow.PNG'),
            PIXI.Texture.from('/assets/images/seven-black.PNG'),
            PIXI.Texture.from('/assets/images/seven-yellow.PNG'),
            PIXI.Texture.from('/assets/images/triple7-black.PNG'),
          ];

          // Build the reels
          // const reels = [];
          const reelContainer = new PIXI.Container();
          reelContainer.y = margin / 2;
          reelContainer.x = margin / 4;
          // 3 reels
          for (let i = 0; i < 3; i++) {
            const rc = new PIXI.Container();
            rc.x = i * REEL_WIDTH + (margin / 2);
            reelContainer.addChild(rc);

            const reel = {
              container: rc,
              symbols: [],
              position: 0,
              previousPosition: 0,
              blur: new PIXI.filters.BlurFilter(),
            };
            reel.blur.blurX = 0;
            reel.blur.blurY = 0;
            rc.filters = [reel.blur];

            // Build the symbols
            for (let j = 0; j < 5; j++) {
              const symbol = new PIXI.Sprite(slotTextures[j]);
              // Scale the symbol to fit symbol area.
              symbol.y = j * SYMBOL_SIZE;
              symbol.scale.x = symbol.scale.y = Math.min(SYMBOL_SIZE / symbol.width, SYMBOL_SIZE / symbol.height);
              symbol.x = Math.round((SYMBOL_SIZE - symbol.width + margin / 2) / 2);
              switch(j) {
                case 0:
                  symbol.name = "one";
                  break;
                case 1:
                  symbol.name = "two";
                  break;
                case 2:
                  symbol.name = "three";
                  break;
                case 3:
                  symbol.name = "four";
                  break;
                case 4:
                  symbol.name = "five";
                  break;
                default:
              }

              reel.symbols.push(symbol);
              rc.addChild(symbol);
            }
            _self.reels.push(reel);
          }

          app.stage.addChild(reelContainer);

          //Borders
          // drawRect(x,y,width,height)
          const leftBorder = new PIXI.Graphics();
          leftBorder.beginFill(0, 1);
          leftBorder.drawRect(0, 0, margin / 2, app.screen.height);

          const rightBorder = new PIXI.Graphics();
          rightBorder.beginFill(0, 1);
          rightBorder.drawRect(app.screen.width - margin / 2, 0, margin, app.screen.height);

          const bottomBorder = new PIXI.Graphics();
          bottomBorder.beginFill(0, 1);
          bottomBorder.drawRect(0, app.screen.height - margin, app.screen.width, margin);

          app.stage.addChild(leftBorder);
          app.stage.addChild(bottomBorder);


          // Top Border
          const top = new PIXI.Graphics();
          top.beginFill(0, 1);
          top.drawRect(0, 0, app.screen.width, margin);

          // Add Jackpot Text
          // const playText = new PIXI.Text("JACKPOT", style);
          // playText.x = Math.round((top.width - playText.width) / 2);
          // playText.y = Math.round((margin - playText.height) / 2);
          // top.addChild(playText);

          // // Add ClickMe Text
          // const clickMe = new PIXI.Text("Click Me");
          // clickMe.style = {fill: "white", fontSize: 40};
          // clickMe.x = Math.round((top.width - clickMe.height));
          // clickMe.y = Math.round((margin + clickMe.width));
          // clickMe.anchor.x = 0.5;
          // clickMe.anchor.y = 0.5;
          // clickMe.angle = 90;
          // rightBorder.addChild(clickMe);


          app.stage.addChild(top);
          app.stage.addChild(rightBorder);


          // Set the interactivity.
          // rightBorder.interactive = true;
          // rightBorder.buttonMode = true;
          // rightBorder.addListener('pointerdown', () => {
          //   this.startPlay();
          // });

          // Listen for animate update.
          app.ticker.add((delta) => {
            // Update the slots.
            for (let i = 0; i < _self.reels.length; i++) {
              const r = _self.reels[i];
              // Update blur filter y amount based on speed.
              // This would be better if calculated with time in mind also. Now blur depends on frame rate.
              r.blur.blurY = (r.position - r.previousPosition) * 8;
              r.previousPosition = r.position;

              // Update symbol positions on reel.
              for (let j = 0; j < r.symbols.length; j++) {
                const s = r.symbols[j];
                const prevy = s.y;
                s.y = ((r.position + j) % r.symbols.length) * SYMBOL_SIZE - SYMBOL_SIZE;
                // if (s.y < 0 && prevy > SYMBOL_SIZE) {
                //   // Detect going over and swap a texture.
                //   // This should in proper product be determined from some logical reel.
                //   s.texture = slotTextures[Math.floor(Math.random() * slotTextures.length)];
                //   s.scale.x = s.scale.y = Math.min(SYMBOL_SIZE / s.texture.width, SYMBOL_SIZE / s.texture.height);
                //   s.x = Math.round((SYMBOL_SIZE - s.width) / 2);
                // }
              }

            }
          });

          // Listen for animate update.
          app.ticker.add((delta) => {
            const now = Date.now();
            const remove = [];
            for (let i = 0; i < _self.tweening.length; i++) {
              const t = _self.tweening[i];
              const phase = Math.min(1, (now - t.start) / t.time);

              t.object[t.property] = lerp(t.propertyBeginValue, t.target, t.easing(phase));
              if (t.change) t.change(t);
              if (phase === 1) {
                t.object[t.property] = t.target;
                if (t.complete) t.complete(t);
                remove.push(t);
              }
            }
            for (let i = 0; i < remove.length; i++) {
              _self.tweening.splice(_self.tweening.indexOf(remove[i]), 1);
            }
          });

          // Basic lerp funtion.
          function lerp(a1, a2, t) {
            return a1 * (1 - t) + a2 * t;
          }

          // Backout function from tweenjs.
          // https://github.com/CreateJS/TweenJS/blob/master/src/tweenjs/Ease.js

        } //END ONASSETSLOADED FUNCTION

      let w;
      let h;
      if (window.innerWidth <= 992) {
        w = window.innerWidth - 100;
        h = window.innerWidth / (800/500) - 100;
      } else {
        w = this.width;
        h = this.height;
      }
      app.renderer.view.style.width = w + 'px';
      app.renderer.view.style.height = h + 'px';

      window.addEventListener("resize", function(){
        let w;
        let h;å
        if (window.innerWidth <= 992) {
          w = window.innerWidth - 100;
          h = window.innerWidth / (800/500) - 100;
        } else {
          w = this.width;
          h = this.height;
        }
        app.renderer.view.style.width = w + 'px';
        app.renderer.view.style.height = h + 'px';
      });
    },
    methods: {
      // Function to start playing.
      startPlay() {
        if (this.running) return;
        this.running = true;

        for (let i = 0; i < this.reels.length; i++) {
          const r = this.reels[i];
          const extra = Math.floor(Math.random() * 3);
          const target = r.position + 10 + i * 5 + extra;
          const time = 2500 + i * 600 + extra * 600;
          this.tweenTo(r, 'position', target, time, this.backout(0.5), null, i === this.reels.length - 1 ? this.reelsComplete : null);

          // GSAP
          let tl = gsap.timeline();
          tl.fromTo("#lever", {rotationX: 0, transformOrigin:"0 90%"}, {rotationX: 65, duration: 0.4, ease: "linear"})
                  .fromTo("#lever", {rotationX: 65, transformOrigin:"0 90%"}, {rotationX: 0, duration: 10, ease: "elastic"})
        }
      },
      reelsComplete() {
        this.running = false;
        this.checkWin()
      },
      checkWin(){
        let array = [];

        this.reels.forEach((reel, index) => {
          this.reels[index].symbols.forEach(symbol => {
            if(symbol.position._y < 131 && symbol.position._y > 129.8) {
              array.push(symbol.name)
            }
          });
        });

        const allEqual = (arr) => arr.every( v => v === arr[0] );

        if(allEqual(array))
        {
          this.coins('body');
        }
      },
      tweenTo(object, property, target, time, easing, onchange, oncomplete) {
        const tween = {
          object,
          property,
          propertyBeginValue: object[property],
          target,
          easing,
          time,
          change: onchange,
          complete: oncomplete,
          start: Date.now(),
        };

        this.tweening.push(tween);
        return tween;
      },
      backout(amount) {
        return (t) => (--t * t * ((amount + 1) * t + amount) + 1);
      },
      coins(el) {
        //http://jsfiddle.net/z3roj35k/1/
        const element = document.querySelector(el);
        let canvas = document.createElement('canvas'),
                ctx = canvas.getContext('2d'),
                focused = false;

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        canvas.id = 'coinrain';

        const coin = new Image();
        coin.src = 'http://i.imgur.com/5ZW2MT3.png';
        // 440 wide, 40 high, 10 states
        coin.onload = function () {
          element.appendChild(canvas);
          focused = true;
          drawloop();
        };
        let coins = [];

        function drawloop() {
          if (focused) {
            requestAnimationFrame(drawloop);
          }

          ctx.clearRect(0, 0, canvas.width, canvas.height);

          if (Math.random() < .3) {
            coins.push({
              x: Math.random() * canvas.width | 0,
              y: -50,
              dy: 3,
              s: 0.5 + Math.random(),
              state: Math.random() * 10 | 0
            })
          }
          let i = coins.length;
          while (i--) {
            let x = coins[i].x;
            let y = coins[i].y;
            let s = coins[i].s;
            let state = coins[i].state;
            coins[i].state = (state > 9) ? 0 : state + 0.1;
            coins[i].dy += 0.3
            coins[i].y += coins[i].dy;

            ctx.drawImage(coin, 44 * Math.floor(state), 0, 44, 40, x, y, 44 * s, 40 * s);

            if (y > canvas.height) {
              coins.splice(i, 1);
            }
          }
        } // END of function
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .lever {
    height: calc(100vw / (800/600) - 175px);
    max-height: 400px;
    width: 18px;
    background: #555555;
    border: 0;
    position: relative;
    cursor: pointer;
    -webkit-border-radius: 25px;
    -moz-border-radius: 25px;
    border-radius: 25px;
    outline: none;
    margin-top: 20px;
    z-index: 10;
  }

  .lever::before {
    content: 'Tap here';
    width: 12vw;
    height: 12vw;
    max-width: 125px;
    max-height: 125px;
    position: absolute;
    -webkit-border-radius: 50%;
    -moz-border-radius: 50%;
    border-radius: 50%;
    top: 0;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #FFCA05;
    box-shadow: inset -25px -15px 40px rgba(0,0,0,.3);
    background-image: linear-gradient(-45deg, rgba(255,255,220,.3) 0%, transparent 100%);
    display: flex;
    justify-content: center;
    align-items: center;
    font-weight: bold;
    z-index: 11;
  }

  @media (hover: hover) {
    .lever::before {
      content: '';
    }
  }

  @media screen and (min-width: 768px) {
    .lever {
      width: 40px;
      margin-top: 40px;
    }
  }
</style>
