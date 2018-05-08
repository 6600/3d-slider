<template>
  <section id="slideshow">
    <div class="entire-content">
      <div v-if="tableData" class="content-carrousel" :class="{rotate: enlargeItem === 0}">
        <figure class="shadow" @click="enlarge(1)" :class="{enlarge: enlargeItem === 1}">
          <Excl :data="tableData"></Excl>
        </figure>
        <figure class="shadow" @click="enlarge(2)" :class="{enlarge: enlargeItem === 2}">
          <Excl :data="tableData2"></Excl>
        </figure>
        <figure class="shadow" @click="enlarge(3)" :class="{enlarge: enlargeItem === 3}">
          <Excl :data="tableData3"></Excl>
        </figure>
      </div>
    </div>
    <div class="close" v-show="enlargeItem !== 0" @click="enlargeItem = 0">x</div>
  </section>
</template>

<script>
import Excl from './components/Excl.vue'
export default {
  name: 'app',
  data () {
    return {
      isPaused: false,
      enlargeItem: 0,
      tableData: null,
      tableData2: null,
      tableData3: null
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
    }
  },
  created () {
    const sendData = {
      type: 'id',
      data: '5'
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
    fetch('http://192.168.1.105/index/index/select_data/', fetchConfig).then((res) => {
      if (res.ok) { // 请求成功执行回掉
        res.json().then((data) => {
          data.data = data.data.replace(/NULL/g, '')
          data.data = JSON.parse(data.data)
          this.tableData = this.deepClone(data)
          this.tableData2 = this.deepClone(data)
          this.tableData3 = this.deepClone(data)
        })
      }
    }, (e) => {
      console.error('[POST]请求失败！', e)
    })
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
    background-color: #6adecd;
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
    perspective: 1000px;
    position: relative;
  }

  .content-carrousel {
    width: 100%;
    height: 100%;
    position: absolute;
    float: right;
  }
  .rotate {
    /* 调节动画快慢 */
    animation: rotar 60s infinite linear;
    transform-style: preserve-3d;
  }

  .entire-content .content-carrousel .enlarge {
    position: fixed;
    width: 100%;
    height: 100%;
    transform: none;
    z-index: 999;
  }
  .content-carrousel .enlarge .title {
    height: 40px;
    line-height: 40px;
  }
  .content-carrousel:hover {
    animation-play-state: paused;
  }

  .content-carrousel figure {
    width: 600px;
    height: 410px;
    z-index: 1;
    transition: all 500ms;
    border: 1px solid #3b444b;
    overflow: hidden;
    position: absolute;
  }

  .content-carrousel figure:nth-child(1) { transform: rotateY(0deg) translateZ(300px); }
  .content-carrousel figure:nth-child(2) { transform: rotateY(120deg) translateZ(300px); }
  .content-carrousel figure:nth-child(3) { transform: rotateY(240deg) translateZ(300px); }
  .shadow {
    cursor: pointer;
    position: absolute;
    box-shadow: 0px 0px 20px 0px #000;
    border-radius: 1px;
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
    height: 40px;
    width: 40px;
    cursor: pointer;
    background-color: khaki;
    text-align: center;
    line-height: 40px;
    font-size: 2rem;
  }
  @keyframes rotar {
    from {
      transform: rotateY(0deg);
    } to {
      transform: rotateY(360deg);
    }
  }
</style>
