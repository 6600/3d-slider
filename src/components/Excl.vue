<template>
  <div class="show shadow">
    <div class="table-box">
      <table v-if="tableData" border="0">
        <tr v-for="(item, ind) in tableData" :key="ind">
          <td v-for="(data, index) in item" :key="index" :rowspan="data.rowspan" :colspan="data.colspan">{{data.value === 'NULL' ? '' : fixed(data.value)}}</td>
        </tr>
      </table>
    </div>
    <div class="left-top border"></div>
    <div class="left-bottom border"></div>
    <div class="right-top border"></div>
    <div class="right-bottom border"></div>
  </div>
</template>

<script> 
export default {
  name: 'excl',
  props: {
    data: Object
  },
  data () {
    return {
      tableData: null
    }
  },
  created () {
    // 将数据转换格式
    let dataCopy = JSON.parse(this.data.data)
    let format = JSON.parse(this.data.format)
    let newData = []
    // console.log(dataCopy)
    for (const ind in dataCopy) {
      // console.log(ind)
      // console.log(dataCopy[ind])
      let columnData = this.deepClone(dataCopy[ind])
      // console.log(columnData)
      for (const key in columnData) {
        // console.log(columnData[key])
        const value = this.deepClone(columnData[key])
        // console.log(value)
        columnData[key] = {
          value,
          colspan: 1,
          rowspan: 1
        }
      }
      // console.log(columnData)
      dataCopy[ind] = columnData
    }
    // 进行行/列合并处理
    for (const key in format) {
      const formatItem = format[key]
      // console.log(formatItem)
      // 拆分出合并的起始和末尾
      const first = formatItem.split(':')
      const start = first[0]
      const end = first[1]
      // 将起始和末尾再分割出行列
      const startCoordinate = start.split('-')
      const startRow = startCoordinate[0]
      const startColumn = startCoordinate[1]
      const endCoordinate = end.split('-')
      const endRow = endCoordinate[0]
      const endColumn = endCoordinate[1]
      // 根据开始和结束行坐标是否相同 判断是行合并还是列合并
      if (startColumn === endColumn) {
        // 行合并
        // console.log(startRow, endRow)
        // console.log(this.betwon(startRow, endRow, 1, 1))
        const blockList = this.betwon(startRow, endRow, 1, 1)
        // 进行行合并
        // 减去1的原因是数组是由0键值开始
        // 加上1的原因是数组中没有包含开始项
        dataCopy[startColumn - 1][startRow].colspan = blockList.length + 1
        // 删除空行
        blockList.forEach(element => {
          delete dataCopy[startColumn - 1][element]
        })
      } else {
        // console.log(dataCopy[startColumn - 1][startRow].rowspan)
        dataCopy[startColumn - 1][startRow].rowspan = parseInt(endColumn) - parseInt(startColumn) + 1
        for (let i = parseInt(startColumn); i < parseInt(endColumn); i++) {
          delete dataCopy[i][startRow]
        }
      }
    }
    this.tableData = dataCopy
    // console.log(this.tableData)
  },
  methods: {
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
    // 去字母排序中间
    betwon (start, end, startNum = 0, endNum = 0) {
      const list = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', ' U', ' V', ' W', ' X', ' Y', 'Z']
      const obj = {
        A: 0,
        B: 1,
        C: 2,
        D: 3,
        E: 4,
        F: 5,
        G: 6,
        H: 7,
        I: 8,
        J: 9,
        K: 10,
        L: 11,
        M: 12,
        N: 13,
        O: 14,
        P: 15,
        Q: 16,
        R: 17,
        S: 18,
        T: 19,
        U: 20,
        V: 21,
        W: 22,
        X: 23,
        Y: 24,
        Z: 25
      }
      return list.slice(obj[start.toUpperCase()] + startNum, obj[end.toUpperCase()] + startNum)
    },
    // 保留两位小数
    fixed (text) {
      if (typeof(text) === 'number' && text.toString().length > 6) {
        return text.toFixed(2)
      }
      else return text
    }
  }
}
</script>


<style scoped>
  .show {
    height: 100%;
    width: 100%;
    cursor: pointer;
    transform: scale(0.6, 0.6);
    /* 数据归类统计 */
    /* transform: scale(0.7, 0.7); */
    background-repeat: no-repeat;
    border-radius: 1px;
  }
  .border {
    position: absolute;
    background-repeat: no-repeat;
    background-size: 320px;
  }
  .table-box {
    width: 100%;
    height: 100%;
    overflow: hidden;
  }
  .left-top {
    width: 30%;
    height: 20%;
    left: -20px;
    top: 0px;
    background-image: url(../assets/left-top.svg);
  }
  .right-top {
    width: 30%;
    height: 20%;
    right: -20px;
    top: 0px;
    background-image: url(../assets/right-top.svg);
  }
  .right-bottom {
    width: 320px;
    height: 143px;
    right: -20px;
    bottom: -20px;
    background-image: url(../assets/right-bottom.svg);
  }
  .left-bottom {
    width: 320px;
    height: 143px;
    left: -20px;
    bottom: -20px;
    background-image: url(../assets/left-bottom.svg);
  }
  table {
    color: white;
    font-weight: 100;
    width: calc(100% - 20px);
    font-size: 12px;
    border-spacing: 0px;
    height: calc(100% - 15px);
  }
  table tr {
    background-size: cover;
    background-repeat: no-repeat;
    background-position-y: center;
    background-image: url(../assets/table.svg);
  }
  table tr:nth-child(1) {
    display: none;
    background-image: none;
  }
  table tr:nth-child(2) td {
    border-bottom: none;
  }
  table tr:nth-child(2) {
    height: 30px;
    color: white;
    font-size: 14px;
    background-image: none;
  }
  table tr:nth-child(3) {
    font-size: 12px;
    background-image: none;
  }
  table tr:nth-child(3) td {
    color: white;
    padding: 0 5px;
    text-align: left;
    font-size: 12px;
  }
  table td {
    text-align: center;
  }
</style>
