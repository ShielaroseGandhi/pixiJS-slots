<template>
  <div>
    <button @click="startPlay">
      SPIN
    </button>
  </div>
</template>

<script>
  import * as PIXI from 'pixi.js'

  export default {
    name: 'Slots',
    data(){
      return {
        reels: [],
        running: false,
        tweening: [],
      }
    },
    props: {
    },
    mounted(){
        const app = new PIXI.Application({
          backgroundColor: 0xefefef
        });

        document.body.appendChild(app.view);

        const REEL_WIDTH = 216;
        const SYMBOL_SIZE = 150;
        const margin = (app.screen.height - SYMBOL_SIZE * 3);

        const style = new PIXI.TextStyle({
          fontFamily: 'Arial',
          fontSize: 75,
          fontWeight: 'bold',
          fill: ['#FFD700', '#ffffff', '#FFD700'], // gradient
          wordWrap: true,
          wordWrapWidth: 440,
        });

        // Build the reels
        // const reels = [];
        const reelContainer = new PIXI.Container();
        reelContainer.y = margin / 2;
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
          for (let j = 0; j < 4; j++) {
            const symbol = new PIXI.Text("7", style);
            // Scale the symbol to fit symbol area.
            symbol.y = j * SYMBOL_SIZE;
            symbol.scale.x = symbol.scale.y = Math.min(SYMBOL_SIZE / symbol.width, SYMBOL_SIZE / symbol.height);
            symbol.x = Math.round((SYMBOL_SIZE - symbol.width + margin / 2) / 2);
            reel.symbols.push(symbol);
            rc.addChild(symbol);
          }
          this.reels.push(reel);
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
        const playText = new PIXI.Text("JACKPOT", style);
        playText.x = Math.round((top.width - playText.width) / 2);
        playText.y = Math.round((margin - playText.height) / 2);
        top.addChild(playText);

        // Add ClickMe Text
        const clickMe = new PIXI.Text("Click Me");
        clickMe.style = {fill: "white", fontSize: 40};
        clickMe.x = Math.round((top.width - clickMe.height));
        clickMe.y = Math.round((margin + clickMe.width));
        clickMe.anchor.x = 0.5;
        clickMe.anchor.y = 0.5;
        clickMe.angle = 90;
        rightBorder.addChild(clickMe);


        app.stage.addChild(top);
        app.stage.addChild(rightBorder);


        // Set the interactivity.
        rightBorder.interactive = true;
        rightBorder.buttonMode = true;
        rightBorder.addListener('pointerdown', () => {
          this.startPlay();
        });

        // Listen for animate update.
        app.ticker.add((delta) => {
          // Update the slots.
          for (let i = 0; i < this.reels.length; i++) {
            const r = this.reels[i];
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
          for (let i = 0; i < this.tweening.length; i++) {
            const t = this.tweening[i];
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
            this.tweening.splice(this.tweening.indexOf(remove[i]), 1);
          }
        });

        // Basic lerp funtion.
        function lerp(a1, a2, t) {
          return a1 * (1 - t) + a2 * t;
        }

        // Backout function from tweenjs.
        // https://github.com/CreateJS/TweenJS/blob/master/src/tweenjs/Ease.js

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
        }
      },
      reelsComplete() {
        this.running = false;
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
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
