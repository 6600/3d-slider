<template>
  <section id="slideshow">
    <div class="entire-content">
      <div class="content-carrousel" :class="{plane: enlargeItem !== 0}" @touchmove="turn" @touchend="clearX" @mouseover="isPaused = true" @mouseout="isPaused = false">
        <figure v-for="(item, ind) in tableData" class="shadow" @click="enlarge(ind + 1)" :style="{transform: 'rotateY(' + (rotation + config.angle * ind) + 'deg) translateZ(' + config.translateZ +'px)'}" :class="{enlarge: enlargeItem === ind + 1}" :key="ind">
          <Excl v-if="tableData" :data="item"></Excl>
        </figure>
      </div>
      <div class="file-panel">
        <div class="file-name" v-for="(item, ind) in tableData" @click="enlarge(ind + 1)" :key="item.id">{{item.name}}</div>
      </div>
      <div class="close" v-show="enlargeItem !== 0" @click="enlargeItem = 0">x</div>
      <div class="update" @click="refash">
        <svg t="1525943602085" class="icon" style="" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="4708" xmlns:xlink="http://www.w3.org/1999/xlink" width="24" height="24"><defs></defs><path d="M962.074 490.554L647.271 355.638l125.831-89.881c-65.56-69.108-157.968-112.493-260.747-112.493-174.244 0-319.503 123.884-352.634 288.369l-83.949-34.777C123.56 209.825 300.628 63.32 512.355 63.32c132.973 0 252.063 58.09 334.393 149.833l115.326-82.375v359.776z m-710.47 269.773c65.556 69.108 157.973 112.488 260.752 112.488 174.918 0 320.546-124.873 352.931-290.307l83.868 35.874c-47.481 197.458-224.77 344.377-436.799 344.377-132.973 0-252.068-58.086-334.398-149.828l-115.322 82.37V535.525L377.44 670.441l-125.836 89.886z" p-id="4709"></path></svg>
      </div>
    </div>
  </section>
</template>

<script>
let startX = null
import Excl from './components/Excl.vue'
export default {
  name: 'app',
  data () {
    return {
      rotation: 360,
      clock: null,
      isPaused: false,
      enlargeItem: 0,
      config: {
        angle: 30,
        translateZ: 650,
        speed: 1,
        requestList: [
          '泰州能效对标日对标(1日).xls',
          '泰州能效对标日对标(2日).xls',
          '泰州能效对标日对标(3日).xls',
          '泰州能效对标日对标(4日).xls',
          '国电泰州机组对标月报表(1月).xlsx',
          '国电泰州机组对标月报表(2月).xlsx',
          '国电泰州机组对标月报表(3月).xlsx',
          '国电泰州机组对标月报表(4月).xlsx',
          '国电泰州#3至#4机组对比报表(1月).xlsx',
          '国电泰州#3至#4机组对比报表(2月).xlsx',
          '国电泰州#3至#4机组对比报表(3月).xlsx',
          '国电泰州#3至#4机组对比报表(4月).xlsx',
          '国电泰州机组对标环比月报表.xlsx',
          '国电泰州机组对标同比月报表.xlsx'
        ]
      },
      tableData: null
    }
  },
  components: {
    Excl
  },
  methods: {
    enlarge (key) {
      this.enlargeItem = key
    },
    // 深拷贝
    deepClone (obj) {
      if (obj == null || typeof obj !== 'object') {
        return obj
      }
      switch (Object.prototype.toString.call(obj)) {
        case '[object Array]': {
          const result = new Array(obj.length)
          for (let i = 0; i < result.length; ++i) {
            result[i] = this.deepClone(obj[i])
          }
          return result
        }
        // Object.prototype.toString.call(new XxxError) returns '[object Error]'
        case '[object Error]': {
          const result = new obj.constructor(obj.message)
          result.stack = obj.stack // hack...
          return result
        }
        case '[object Date]':
        case '[object RegExp]':
        case '[object Int8Array]':
        case '[object Uint8Array]':
        case '[object Uint8ClampedArray]':
        case '[object Int16Array]':
        case '[object Uint16Array]':
        case '[object Int32Array]':
        case '[object Uint32Array]':
        case '[object Float32Array]':
        case '[object Float64Array]':
        case '[object Map]':
        case '[object Set]':
          return new obj.constructor(obj)
        case '[object Object]': {
          const keys = Object.keys(obj)
          const result = {}
          for (let i = 0; i < keys.length; ++i) {
            const key = keys[i]
            result[key] = this.deepClone(obj[key])
          }
          return result
        }
        default: {
          throw new Error("Unable to copy obj! Its type isn't supported.")
        }
      }
    },
    animate () {
      if (!this.isPaused  && this.enlargeItem === 0) {
        if (this.rotation > 360) this.rotation = 0
        if (this.rotation < 0) this.rotation = 360
        this.rotation -= this.config.speed / 10
      }
      requestAnimationFrame(this.animate)
    },
    turn (e) {
      const touchX = event.targetTouches[0].pageX
      if (startX !== null) {
        const change = startX - touchX
        this.rotation -= change / 5
      }
      startX = touchX
    },
    clearX () {
      startX = null
    },
    refash () {
      // http://172.103.1.221:8081/excel/public/index.php?s=index/index/
      fetch('http://172.103.1.221:8081/excel/public/index.php?s=index/index/filedir').then((res) => {
        if (res.ok) { // 请求成功执行回掉
          res.json().then((data) => {
            console.log(data)
          })
        }
      }, (e) => {
        alert('刷新失败!')
        console.error('[POST]请求失败！', e)
      })
    }
  },
  created () {
    const sendData = {
      type: 'name',
      data: this.config.requestList.toString()
    }
    // 发起post请求
    const fetchConfig = {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
      },
      credentials: 'credentials',
      cache: 'default',
      body: JSON.stringify(sendData)
    }
    fetch('http://openvticn.com/excel/public/index.php?s=index/index/select_data', fetchConfig).then((res) => {
      if (res.ok) { // 请求成功执行回掉
        res.json().then((data) => {
          // data.data = JSON.parse(data.data)
          this.tableData = this.deepClone(data)
        })
      }
    }, (e) => {
      alert('从服务器获取数据失败!')
      console.error('[POST]请求失败！', e)
    })
  },
  mounted () {
    // 自动旋转
    this.animate()
  }
}
</script>

<style>
  * {
    margin: 0;
    padding: 0;
  }

  html, body {
    max-width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
    overflow-x: hidden;
    overflow-y: auto;
    transform: translate3d(0, 0, 0);
  }

  #slideshow {
    margin: 0 auto;
    height: 350px;
    width: 100%;
    height: 100%;
    background-color: #35373e;
    box-sizing: border-box;
  }

  .slideshow-title {
    font-family: 'Allerta Stencil';
    font-size: 62px;
    color: #fff;
    margin: 0 auto;
    text-align: center;
    margin-top: 25%;
    letter-spacing: 3px;
    font-weight: 300;
  }

  .sub-heading {
    padding-top: 50px;
    font-size: 18px;
  }
  .sub-heading-two { font-size: 15px; }
  .sub-heading-three { font-size: 13px; }
  .sub-heading-four { font-size: 11px; }
  .sub-heading-five { font-size: 9px; }
  .sub-heading-six { font-size: 7px; }
  .sub-heading-seven { font-size: 5px; }
  .sub-heading-eight { font-size: 3px; }
  .sub-heading-nine { font-size: 1px; }

  .entire-content {
    margin: auto;
    width: 100%;
    height: 100%;
    overflow: hidden;
    perspective: 1000px;
    position: relative;
  }

  .content-carrousel {
    width: 100%;
    height: 100%;
    position: absolute;
    transform-style: preserve-3d;
  }
  .plane {
    transform-style: unset;
  }
  .plane figure {
    transform: none !important;
  }

  .entire-content .content-carrousel .enlarge {
    position: fixed;
    width: 100%;
    height: 100%;
    transform: none;
    z-index: 99;
  }
  .content-carrousel .enlarge .title {
    height: 40px;
    line-height: 40px;
  }
  .content-carrousel .enlarge table {
    height: unset;
  }
  .content-carrousel .enlarge table tr:nth-child(2) {
    height: 40px;
    line-height: 40px;
  }
  .content-carrousel .enlarge td {
    height: 20px;
    font-size: 14px;
  }
  .content-carrousel .enlarge .show {
    transform: none;
  }
  .content-carrousel figure {
    transform: scale(2,2);
    width: 540px;
    height: 380px;
    z-index: 1;
    overflow: hidden;
    position: absolute;
  }

  .content-carrousel figure:nth-child(1) { transform: rotateY(0deg) translateZ(300px); }
  .content-carrousel figure:nth-child(2) { transform: rotateY(120deg) translateZ(300px); }
  .content-carrousel figure:nth-child(3) { transform: rotateY(240deg) translateZ(300px); }
  .shadow {
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    margin: auto;
  }
  .close {
    position: fixed;
    right: 0;
    top: 0;
    z-index: 100;
    height: 29px;
    width: 29px;
    cursor: pointer;
    background-color: burlywood;
    text-align: center;
    line-height: 29px;
    font-size: 2rem;
  }
  .file-panel {
    position: fixed;
    top: 5px;
    right: 5px;
    background-color: #454c5a;
    opacity: 0.8;
  }
  .file-panel .file-name {
    color: white;
    padding: 0 5px;
    height: 25px;
    cursor: pointer;
    line-height: 25px;
  }
  .file-panel .file-name:hover {
    color: black;
    background-color: darkkhaki;
  }
  .update {
    position: fixed;
    right: 10px;
    bottom: 10px;
  }
  .update svg {
    fill: cadetblue;
  }
  .update svg:hover {
    fill: antiquewhite;
  }
  @keyframes rotar {
    from {
      transform: rotateY(0deg);
    } to {
      transform: rotateY(360deg);
    }
  }
</style>
