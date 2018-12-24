<template>
  <div id="app">
    <div class="svg-wrapper">
      <svg class="svg" viewBox="0 0 1000 1000">
        <circle :r="clockRadius" :cx="clockRadius" :cy="clockRadius" class="clock"/>

        <path class="arrow" :d="`M${minutePositionA.join(',')} Q${pointPositionA.join(',')} ${centerPositionA.join(',')} T${hourPosition.join(',')}`"/>

        <template v-if="debugDrawings">
          <circle class="help" :r="minuteRadius" cx="500" cy="500"/>
          <circle class="help" :r="hourRadius" cx="500" cy="500"/>
          <circle class="help" :r="centerRadius" cx="500" cy="500"/>
          <circle class="center" r="5" cx="500" cy="500"/>
          <circle class="center" r="5" :cx="centerPositionA[0]" :cy="centerPositionA[1]"/>
          <circle class="center" r="5" :cx="pointPositionA[0]" :cy="pointPositionA[1]"/>
        </template>
      </svg>
      <!--<input type="range" min="0" :max="clockRadius" v-model.number="minuteRadius" style="width: 100%"/>-->
      <!--<input type="range" min="0" :max="clockRadius" v-model.number="hourRadius" style="width: 100%"/>-->
      <!--<input type="range" min="0" :max="clockRadius" v-model.number="centerRadius" style="width: 100%"/>-->
      <!--<input type="range" min="0" :max="clockRadius" v-model.number="pointLength" style="width: 100%"/>-->
    </div>
    <footer class="footer">
      <span>Made with ❤️ by <a href="https://loskir.ru" target="_blank">@Loskir</a></span>
      <span><a href="https://github.com/Loskir/flexible-clock" target="_blank">GitHub</a></span>
    </footer>
    <div class="dev-menu" :class="{opened: devMenuOpened}">
      <button class="title" @click="devMenuOpened = !devMenuOpened">Dev menu</button>
      <div class="content">
        <label><input type="checkbox" v-model="debugDrawings"/>Debug drawings</label>
        <input type="number" min="0" max="3600" step="0.1" v-model.number="minute" style="width: 100%"/>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'app',
    data() {
      return {
        clockRadius: 500,
        minuteRadius: 450,
        hourRadius: 250,
        centerRadius: 100,
        pointLength: 100,
        slowAnimationDuration: 2000,
        fastAnimationDuration: 500,

        minute: 15,

        debugDrawings: false,

        timeTimeout: undefined,

        devMenuOpened: false
      }
    },
    methods: {
      startInterval() {
        clearTimeout(this.timeTimeout);
        if (!this.devMenuOpened) {
          console.log('time_update, next in', 1000-Date.now()%1000, 'ms');
          this.timeTimeout = setTimeout(() => this.startInterval(), 1000-Date.now()%1000);
          this.minute = Date.now()/60000%3600
        }
      }
    },
    computed: {
      minuteAngle() {
        return (this.minute%60)/30*Math.PI
      },
      hourAngle() {
        return ((this.minute/30)%60)/30*Math.PI
      },
      centerAngle() {
        let a = this.minuteAngle+this.hourAngle;
        return Math.abs(this.minuteAngle-this.hourAngle) < Math.PI ? a/2+Math.PI : a/2;
      },
      pointAngle() {
        return this.centerAngle + Math.PI*0.5;
      },
      minutePosition() {
        let a = this.minuteAngle;
        let r = this.minuteRadius;
        let c = this.clockRadius;
        return [c+r*Math.sin(a), c-r*Math.cos(a)]
      },
      hourPosition() {
        let a = this.hourAngle;
        let r = this.hourRadius;
        let c = this.clockRadius;
        return [c+r*Math.sin(a), c-r*Math.cos(a)]
      },
      centerPosition() {
        let a = this.centerAngle;
        let r = this.centerRadius;
        let c = this.clockRadius;
        return [c+r*Math.sin(a), c-r*Math.cos(a)]
      },
      pointPosition() {
        let a = this.pointAngle;
        let r = this.pointLength;
        let c = this.centerPosition;

//        let angleDelta = ( Math.abs(this.minuteAngle-this.hourAngle) + 2*Math.PI ) % (2*Math.PI); // [0 : 2PI]
        let angleDelta2 = ( this.minuteAngle-this.hourAngle + 2*Math.PI ) % (2*Math.PI); // [0 : 2PI]
//        console.log(angleDelta2)
        let shouldReverse = angleDelta2 < Math.PI/3;
        let isOnReversedSide = angleDelta2 > Math.PI || angleDelta2 < 0;
//        console.log(shouldReverse, isOnReversedSide);

        let cw = shouldReverse||isOnReversedSide;


        return [c[0]+r*Math.sin(a)*(cw ? 1 : -1), c[1]-r*Math.cos(a)*(cw ? 1 : -1)];
//        return [c[0]-r*Math.sin(a), c[1]+r*Math.cos(a)]
      }
    },
    tweened: {
      minutePositionA: {
        get() {
          return this.minutePosition;
        },
        duration() {
          return this.fastAnimationDuration;
        },
        easing(t) {
          return t * (2 - t);
        }
      },
      centerPositionA: {
        get() {
          return this.centerPosition;
        },
        duration() {
          return this.slowAnimationDuration;
        },
        easing(t) {
          return t * (2 - t);
        }
      },
      pointPositionA: {
        get() {
          return this.pointPosition;
        },
        duration() {
          return this.slowAnimationDuration;
        },
        easing(t) {
          return t * (2 - t);
        }
      }
    },
    watch: {
      minute(val) {
        return val %3600;
      },
      devMenuOpened(val) {
        if (!val) this.startInterval();
      }
    },
    mounted() {
      this.startInterval();
//      setInterval(() => {
//        this.minute++
//      }, 1000)
    }
  }
</script>

<style lang="less">
  @dark-color: #0b0b0b;

  body {
    margin: 0;
  }
  #app {
    font-family: monospace;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: 100vh;
    overflow: hidden;

    a {
      color: #007bff;
      text-decoration: none;
      transition: .2s;
      &:hover {
        color: #0056b3
      }
    }

    .svg-wrapper {
      padding: 100px 50px;
      width: 100%;
      height: 100%;
      box-sizing: border-box;
    }
    .svg {
      max-width: 100%;
      max-height: 100%;

      .clock {
        fill: #e6e6e6
      }
      .arrow {
        stroke: @dark-color;
        fill: none;
        stroke-width: 40;
        stroke-linejoin: round;
      }

      .center {
        fill: red;
      }
      .help {
        stroke: @dark-color;
        stroke-width: 2;
        stroke-dasharray: 5 5;
        fill: none
      }
    }
    .footer {
      position: absolute;
      z-index: 100;
      width: 100%;
      bottom: 0;
      background: white;
      border-top: solid 1px #ced4da;
      padding: 20px 50px;
      box-sizing: border-box;
      display: flex;
      flex-direction: row;
      justify-content: space-between;
    }
    .dev-menu {
      position: absolute;
      z-index: 100;
      right: 0;
      top: 0;

      border: solid 1px #ced4da;
      border-top: none;
      border-right: none;

      transition: .5s;
      width: 80px;

      display: flex;
      flex-direction: column;

      .title {
        cursor: pointer;
        border: none;
        outline: none;
        color: inherit;
        font: inherit;
        background: transparent;

        text-align: center;
        display: inline-block;
        padding: 10px;
        width: 100%;
      }
      .content {
        height: 0;
        overflow: hidden;
        transition: .5s;
      }
      &.opened {
        width: 150px;
        .content {
          height: 50px;
        }
      }
    }
  }
</style>
