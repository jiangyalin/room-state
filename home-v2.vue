<!--300w数量级-->
<template>
  <div class="home">
    <section class="u-box">
      <div class="u-day">
        <ul
          class="u-list"
          :style="dayStyle"
        >
          <li
            v-for="(item, i) in day"
            :key="i"
            class="u-li"
          >
            {{ item.text }}
          </li>
        </ul>
      </div>
      <div class="u-mn">
        <div class="u-room">
          <ul
            class="u-list"
            :style="'transform: translateY(' + scrollTop + 'px)'"
          >
            <li
              v-for="(item, i) in roomListArray"
              :key="i"
              class="u-li"
              :style="item.style"
            >
              {{ item.text }}
            </li>
          </ul>
        </div>
        <div class="u-room-state j-box">
          <ul
            class="u-list j-list"
            :style="boxStyle"
          >
            <li
              v-for="(item, i) in roomStateArray"
              :key="i"
              class="u-li"
              :style="item.style"
            >
              <p>{{ item.index }}</p>
              <p>{{ item.text }}</p>
              <p>{{ item.text }}</p>
              <p>{{ item.text }}</p>
              <!--<p>{{item.text}}</p>-->
              <!--<p>{{item.text}}</p>-->
              <!--<p>{{item.text}}</p>-->
              <!--<p>{{item.text}}</p>-->
              <!--<p style="display: none">{{item.text}}</p>-->
              <!--<p style="display: none">{{item.text}}</p>-->
              <!--<p style="display: none">{{item.text}}</p>-->
              <!--<p style="display: none">{{item.text}}</p>-->
            </li>
          </ul>
        </div>
      </div>
    </section>
  </div>
</template>

<script type="text/ecmascript-6">
export default {
  name: 'home',
  components: {},
  props: [],
  data () {
    return {
      w: 6000, // 列
      h: 500, // 行
      margin: 1, // 冗余量
      day: [],
      roomList: [],
      roomState: [],
      roomStateMap: {},
      box: {
        boxTop: 0,
        boxBottom: 0,
        boxLeft: 0,
        boxRight: 0
      },
      scrollTop: 0,
      scrollLeft: 0,
      roomListLast: {
        lastTop: 0,
        lastBottom: 0
      },
      roomStateListLast: {
        lastTop: -1,
        lastLeft: 0,
        lastBottom: 0,
        lastRight: 0
      },
      boxDome: {},
      xSize: '',
      ySize: '',
      roomStateArray: [],
      isScroll: false // 是否正在滚动中
    }
  },
  computed: {
    roomListArray () {
      return this.roomList.filter(item => item.show && !this.isScroll)
    },
    boxStyle () {
      return {
        width: this.w * 96 + 'px',
        height: this.h * 48 + 'px'
      }
    },
    dayStyle () {
      return {
        width: (this.w + 2) * 96 + 'px',
        transform: 'translateX(' + this.scrollLeft + 'px)'
      }
    }
  },
  created () {
    console.log('总数据量：', this.w * this.h / 10000 + 'w')
    for (let i = 0; i < this.w; i++) {
      this.day.push({
        text: i,
        style: {
          transform: 'translateX(' + (i % this.w) * 96 + 'px' + ')'
        },
        left: i * 96,
        show: true,
        index: i
      })
    }
    for (let i = 0; i < this.h; i++) {
      this.roomList.push({
        text: '这是名称' + i,
        style: {
          transform: 'translateY(' + i * 48 + 'px' + ')'
        },
        top: i,
        show: false,
        index: i
      })
    }
    const margin = this.margin // 冗余量
    for (let i = 0; i < ((this.w + margin * 2) * (this.h + margin * 2)); i++) {
      const _w = i % (this.w + margin * 2) - margin
      const _h = parseInt(i / (this.w + margin * 2)) - margin
      this.roomStateMap[_h + '-' + _w] = {
        text: _h + '-' + _w,
        style: {
          transform: 'translate(' + _w * 96 + 'px,' + _h * 48 + 'px' + ')'
        },
        left: _w,
        top: _h,
        show: false,
        index: i
      }
      // this.roomState.push({
      //   text: (i % (this.w + margin * 2) - margin) + '-' + (parseInt(i / (this.w + margin * 2)) - margin),
      //   style: {
      //     transform: 'translate(' + _w * 96 + 'px,' + _h * 48 + 'px' + ')'
      //   },
      //   left: i % (this.w + margin * 2) - margin,
      //   top: parseInt(i / (this.w + margin * 2)) - margin,
      //   show: false,
      //   index: i
      // })
    }
  },
  mounted () {
    const dome = document.querySelector('.j-box')
    this.boxDome = dome
    // this.xSize = Math.ceil(dome.clientWidth / 96)
    // this.ySize = Math.ceil(dome.clientHeight / 48)
    this.xSize = Math.ceil(976 / 96)
    this.ySize = Math.ceil(948 / 48)
    console.log('xSize', this.xSize)
    console.log('ySize', this.ySize)
    this.calculationRoomList(0, this.ySize)
    this.calculationInitRoomStateList(0, 0, this.ySize, this.xSize, this.margin)
    // this.roomStateArray = this.roomState.filter(item => item.show)
    dome.onscroll = () => {
      this.isScroll = false
      this.scrollTop = -dome.scrollTop
      this.scrollLeft = -dome.scrollLeft
      const _left = parseInt(-this.scrollLeft / 96)
      const _top = parseInt(-this.scrollTop / 48)
      this.calculationRoomList(_top, this.ySize)
      this.calculationRoomStateList(_top, _left, this.ySize, this.xSize, this.margin)
      this.isScroll = false
    }
    // this.globalResize(() => {
    //   this.xSize = Math.ceil(dome.clientWidth / 96)
    //   this.ySize = Math.ceil(dome.clientHeight / 48)
    //   const _left = parseInt(-this.scrollLeft / 96)
    //   const _top = parseInt(-this.scrollTop / 48)
    //   this.calculationRoomList(_top, this.ySize)
    //   this.calculationInitRoomStateList(_top, _left, this.ySize, this.xSize, 1)
    // })
  },
  methods: {
    // 防抖
    debounce (func, wait, notDebounce, immediate) {
      let timeout = {}
      return () => {
        notDebounce()
        const context = this
        const args = arguments
        const later = () => {
          timeout = null
          if (!immediate) func.apply(context, args)
        }
        const callNow = immediate && !timeout
        clearTimeout(timeout)
        timeout = setTimeout(later, wait)
        if (callNow) func.apply(context, args)
      }
    },
    // 节流
    throttle (func, wait, mustRun) {
      let timeout
      let startTime = new Date()

      return () => {
        const context = this
        const args = arguments
        const curTime = new Date()

        clearTimeout(timeout)
        // 如果达到了规定的触发时间间隔，触发 handler
        if (curTime - startTime >= mustRun) {
          func.apply(context, args)
          startTime = curTime
          // 没达到触发间隔，重新设定定时器
        } else {
          timeout = setTimeout(func, wait)
        }
      }
    },
    /*
    * 计算显示区域时先算显示，然后异步计算隐藏
    * */
    calculationRoomList (top, ySize) {
      const bottom = top + ySize
      // 清除上一次的显示区域
      for (let i = this.roomListLast.lastTop; i < this.roomListLast.lastBottom; i++) {
        this.roomList[i].show = false
      }
      // 显示当前可显示区域
      for (let i = top; i < bottom; i++) {
        this.roomList[i].show = true
      }
      this.roomListLast.lastTop = top
      this.roomListLast.lastBottom = bottom
    },
    calculationInitRoomStateList (top, left, ySize, xSize, margin) {
      const right = left + xSize
      const bottom = top + ySize
      this.roomStateListLast.lastTop = 0
      // 清除上一次的显示区域
      for (let i = this.roomStateListLast.lastTop; i < this.roomStateListLast.lastBottom + margin * 2; i++) {
        for (let j = this.roomStateListLast.lastLeft; j < this.roomStateListLast.lastRight + margin * 2; j++) {
          // this.roomState[i * (this.w + margin * 2) + j].show = false
          this.roomStateMap[i - margin + '-' + j].show = false
        }
      }
      // 显示当前可显示区域
      for (let i = top; i < bottom + margin * 2; i++) {
        for (let j = left; j < right + margin * 2; j++) {
          // this.roomState[i * (this.w + margin * 2) + j].show = true
          this.roomStateMap[i - margin + '-' + j].show = true
        }
      }
      // this.roomStateArray = this.roomState.filter(item => item.show)
      const arr = []
      for (const key in this.roomStateMap) {
        if (this.roomStateMap[key].show) arr.push(this.roomStateMap[key])
      }
      this.roomStateArray = arr
      this.roomStateListLast.lastTop = top
      this.roomStateListLast.lastLeft = left
      this.roomStateListLast.lastBottom = bottom
      this.roomStateListLast.lastRight = right
    },
    calculationRoomStateList (top, left, ySize, xSize, margin) {
      const right = left + xSize
      const bottom = top + ySize
      // 后减前
      const yBeforeToLast = top - this.roomStateListLast.lastTop
      const xBeforeToLast = left - this.roomStateListLast.lastLeft
      // 是否存在重合区域(不存在)
      if (Math.abs(yBeforeToLast) > ySize || Math.abs(xBeforeToLast) > xSize) {
        this.calculationInitRoomStateList(top, left, ySize, xSize, margin)
        return false
      }
      const maxLength = (ySize + margin * 2) * (xSize + margin * 2)
      for (let i = 0; i < maxLength; i++) {
        const reverseIndex = maxLength - i - 1
        // 往下
        if (this.roomStateArray[i].top < (top - margin)) {
          // this.roomStateArray.splice(i, 1, this.roomState[this.roomStateArray[i].index + (ySize + margin * 2) * (this.w + margin * 2)])
          this.roomStateArray.splice(i, 1, this.roomStateMap[(this.roomStateArray[i].top + (ySize + margin * 2)) + '-' + this.roomStateArray[i].left])
        }
        // 往上
        if (this.roomStateArray[reverseIndex].top > (top + (ySize - 1) + margin)) {
          // this.roomStateArray.splice(reverseIndex, 1, this.roomState[(this.roomStateArray[reverseIndex].index - (ySize + margin * 2) * (this.w + margin * 2))])
          this.roomStateArray.splice(reverseIndex, 1, this.roomStateMap[(this.roomStateArray[reverseIndex].top - (ySize + margin * 2)) + '-' + this.roomStateArray[reverseIndex].left])
        }
        // 往右
        if (this.roomStateArray[i].left < left - margin) {
          // this.roomStateArray.splice(i, 1, this.roomState[this.roomStateArray[i].index + xSize + margin * 2])
          this.roomStateArray.splice(i, 1, this.roomStateMap[this.roomStateArray[i].top + '-' + (this.roomStateArray[i].left + (xSize + margin * 2))])
        }
        // 往左
        if (this.roomStateArray[reverseIndex].left > (left + (xSize - 1) + margin)) {
          // this.roomStateArray.splice(reverseIndex, 1, this.roomState[this.roomStateArray[reverseIndex].index - (xSize + margin * 2)])
          this.roomStateArray.splice(reverseIndex, 1, this.roomStateMap[this.roomStateArray[reverseIndex].top + '-' + (this.roomStateArray[reverseIndex].left - (xSize + margin * 2))])
        }
      }
      this.roomStateListLast.lastTop = top
      this.roomStateListLast.lastLeft = left
      this.roomStateListLast.lastBottom = bottom
      this.roomStateListLast.lastRight = right
      // this.roomStateArray = this.roomState.filter(item => item.show)
    }
  }
}
</script>

<style lang="scss" scoped>
::-webkit-scrollbar {
  width: 1px;
  height: 1px;
}

.home {
  margin: 0 auto;
  padding: 20px;
  width: 1200px;
  background-color: #fff;
}

.u-box {
  display: flex;
  flex-direction: column;
  position: relative;
  //width: 90%;
  height: 1000px;
  border: 1px solid #057bff;
  box-sizing: border-box;

  .u-day {
    overflow-x: hidden;
    width: 100%;
    height: 50px;

    .u-list {
      display: flex;
      position: relative;
      padding-left: 96 * 2px;
      width: 96 * 30px + 96 * 2px;
      height: 50px;
    }

    .u-li {
      width: 96px;
      height: 50px;
      text-align: center;
      line-height: 48px;
      border: 1px solid #057bff;
      box-sizing: border-box;
    }
  }

  .u-mn {
    display: flex;
    //flex: 1;
    height: calc(100% - 50px);

    .u-room {
      overflow: hidden;
      width: 191px;
      height: 100%;
      border-right: 1px solid #606266;
      border-top: 1px solid #606266;

      .u-list {
        position: relative;
        width: 100%;
        height: 48 * 500px;
        background-image: url("./../../assets/test/room-bg.png");

        .u-li {
          position: absolute;
          width: 100%;
          height: 48px;
          line-height: 48px;
          border-bottom: 1px solid #606266;
          background-color: #fff;
          box-sizing: border-box;
        }
      }
    }

    .u-room-state {
      overflow: auto;
      flex: 1;
      border: 1px solid #057bff;

      .u-list {
        overflow: hidden;
        position: relative;
        width: 96 * 30px;
        height: 48 * 500px;
        background-image: url("./../../assets/test/room.png");

        .u-li {
          overflow: hidden;
          position: absolute;
          top: 0;
          left: 0;
          width: 96px;
          height: 48px;
          text-align: center;
          border-top: 1px solid #e8e8e8;
          border-left: 1px solid #e8e8e8;
          background-color: #fff;
        }
      }
    }
  }
}
</style>
