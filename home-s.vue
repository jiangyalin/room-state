<template>
  <div class="home">
    <section class="u-box">
      <div class="u-day">
        <ul class="u-list" :style="'transform: translateX(' + scrollLeft + 'px)'">
          <li class="u-li" v-for="(item, i) in day" :key="i">{{item.text}}</li>
        </ul>
      </div>
      <div class="u-mn">
        <div class="u-room">
          <ul class="u-list" :style="'transform: translateY(' + scrollTop + 'px)'">
            <li class="u-li" v-for="(item, i) in roomListArray" :key="i" :style="item.style">{{item.text}}</li>
          </ul>
        </div>
        <div class="u-room-state j-box">
          <ul class="u-list j-list">
            <li class="u-li" v-for="(item, i) in roomStateArray" :key="i" :style="item.style">
              <p>{{item.index}}</p>
              <p>{{item.text}}</p>
              <p>{{item.text}}</p>
              <p>{{item.text}}</p>
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
  data () {
    return {
      day: [],
      roomList: [],
      roomState: [],
      margin: 20,
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
  props: [],
  components: {},
  computed: {
    roomListArray () {
      return this.roomList.filter(item => item.show && !this.isScroll)
    }
  },
  methods: {
    // 检测当前节点是否可见
    elementIsVisibleInViewport (el, sectionIsVisible = true) {
      // const box = document.querySelector('.j-box')
      const top = el.getBoundingClientRect().top
      const bottom = el.getBoundingClientRect().bottom
      const left = el.getBoundingClientRect().left
      const right = el.getBoundingClientRect().right
      // const boxTop = box.getBoundingClientRect().top
      // const boxBottom = box.getBoundingClientRect().bottom
      // const boxLeft = box.getBoundingClientRect().left
      // const boxRight = box.getBoundingClientRect().right
      const boxTop = this.box.top
      const boxBottom = this.box.bottom
      const boxLeft = this.box.left
      const boxRight = this.box.right
      if (sectionIsVisible) {
        /*
        *      +y轴部分可见              -x轴部分可见              -y轴部分可见              +x轴部分可见
        * */
        return bottom - boxTop >= 0 && right - boxLeft >= 0 && boxBottom - top >= 0 && boxRight - left >= 0
      } else {
        /*
        *      +y轴完全可见           -x轴完全可见             -y轴完全可见                 +x轴完全可见
        * */
        return top - boxTop >= 0 && left - boxLeft >= 0 && boxBottom - bottom >= 0 && boxRight - right >= 0
      }
    },
    // 防抖
    debounce (func, wait, notDebounce, immediate) {
      let timeout = {}
      return () => {
        notDebounce()
        let context = this
        let args = arguments
        let later = () => {
          timeout = null
          if (!immediate) func.apply(context, args)
        }
        let callNow = immediate && !timeout
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
        let context = this
        let args = arguments
        let curTime = new Date()

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
    calculationDisplay () {
      document.querySelectorAll('.j-li').forEach(item => {
        const index = item.getAttribute('data-index')
        const isVisible = this.elementIsVisibleInViewport(item)
        this.day = this.day.map(node => {
          return {
            ...node,
            visible: node.index === Number(index) ? isVisible : node.visible
          }
        })
        this.day = this.day.map((node, i) => {
          let show = false
          if (!node.visible) {
            const before = i - this.margin
            const after = i + this.margin
            if (before >= 0) {
              if (this.day[before].visible) show = true
            } else {
              if (this.day[after].visible) show = true
            }
            if (after <= this.day.length - 1) {
              if (this.day[after].visible) show = true
            } else {
              if (this.day[before].visible) show = true
            }
          } else {
            show = true
          }
          return {
            ...node,
            show
          }
        })
      })
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
          this.roomState[i * (30 + margin * 2) + j].show = false
        }
      }
      // 显示当前可显示区域
      for (let i = top; i < bottom + margin * 2; i++) {
        for (let j = left; j < right + margin * 2; j++) {
          this.roomState[i * (30 + margin * 2) + j].show = true
        }
      }
      this.roomStateArray = this.roomState.filter(item => item.show)
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
      } else {
        // const yState = yBeforeToLast >= 0
        // const xState = xBeforeToLast >= 0
        // const y = [this.roomStateListLast.lastTop, this.roomStateListLast.lastBottom, top, bottom]
        // y.sort((a, b) => a - b)
        // const x = [this.roomStateListLast.lastLeft, this.roomStateListLast.lastRight, left, right]
        // x.sort((a, b) => a - b)
        // for (let i = y[1]; i < y[2]; i++) {
        //   for (let j = x[0]; j < x[1]; j++) {
        //     this.roomState[i * 30 + j].show = !xState
        //   }
        //   for (let j = x[2]; j < x[3]; j++) {
        //     this.roomState[i * 30 + j].show = xState
        //   }
        // }
        // // 滚动方向 前左上，后右下  或  前右下，后左上
        // if ((xState && yState) || (!xState && !yState)) {
        //   for (let i = y[0]; i < y[1]; i++) {
        //     for (let j = x[0]; j < x[2]; j++) {
        //       this.roomState[i * 30 + j].show = !yState
        //     }
        //   }
        //   for (let i = y[2]; i < y[3]; i++) {
        //     for (let j = x[1]; j < x[3]; j++) {
        //       this.roomState[i * 30 + j].show = yState
        //     }
        //   }
        //   // 前右上，后左下        或         前左下，后右上
        // } else if ((!xState && yState) || (xState && !yState)) {
        //   for (let i = y[0]; i < y[1]; i++) {
        //     for (let j = x[1]; j < x[3]; j++) {
        //       this.roomState[i * 30 + j].show = !yState
        //     }
        //   }
        //   for (let i = y[2]; i < y[3]; i++) {
        //     for (let j = x[0]; j < x[2]; j++) {
        //       this.roomState[i * 30 + j].show = yState
        //     }
        //   }
        // }

        const maxLength = (ySize + margin * 2) * (xSize + margin * 2)
        for (let i = 0; i < maxLength; i++) {
          const reverseIndex = maxLength - i - 1
          // 往下
          if (this.roomStateArray[i].top < (top - margin)) {
            this.roomStateArray.splice(i, 1, this.roomState[this.roomStateArray[i].index + (ySize + margin * 2) * (30 + margin * 2)])
          }
          // 往上
          if (this.roomStateArray[reverseIndex].top > (top + (ySize - 1) + margin)) {
            this.roomStateArray.splice(reverseIndex, 1, this.roomState[(this.roomStateArray[reverseIndex].index - (ySize + margin * 2) * (30 + margin * 2))])
          }
          // 往右
          if (this.roomStateArray[i].left < left - margin) {
            this.roomStateArray.splice(i, 1, this.roomState[this.roomStateArray[i].index + xSize + margin * 2])
          }
          // 往左
          if (this.roomStateArray[reverseIndex].left > (left + (xSize - 1) + margin)) {
            this.roomStateArray.splice(reverseIndex, 1, this.roomState[this.roomStateArray[reverseIndex].index - (xSize + margin * 2)])
          }
        }
        this.roomStateListLast.lastTop = top
        this.roomStateListLast.lastLeft = left
        this.roomStateListLast.lastBottom = bottom
        this.roomStateListLast.lastRight = right
      }
      // this.roomStateArray = this.roomState.filter(item => item.show)
    }
  },
  created () {
    for (let i = 0; i < 30; i++) {
      this.day.push({
        text: i,
        style: {
          transform: 'translateX(' + (i % 30) * 96 + 'px' + ')'
        },
        left: i * 96,
        show: true,
        index: i
      })
    }
    for (let i = 0; i < 500; i++) {
      this.roomList.push({
        text: '这是房源名称' + i,
        style: {
          transform: 'translateY(' + i * 48 + 'px' + ')'
        },
        top: i,
        show: false,
        index: i
      })
    }
    const margin = 1
    for (let i = 0; i < ((30 + margin * 2) * (500 + margin * 2)); i++) {
      this.roomState.push({
        text: (i % (30 + margin * 2) - margin) + '-' + (parseInt(i / (30 + margin * 2)) - margin),
        style: {
          transform: 'translate(' + (i % (30 + margin * 2) - margin) * 96 + 'px,' + (parseInt(i / (30 + margin * 2)) - margin) * 48 + 'px' + ')'
        },
        left: i % (30 + margin * 2) - margin,
        top: parseInt(i / (30 + margin * 2)) - margin,
        show: false,
        index: i
      })
    }
  },
  mounted () {
    const dome = document.querySelector('.j-box')
    this.boxDome = dome
    this.xSize = Math.ceil(dome.clientWidth / 96)
    this.ySize = Math.ceil(dome.clientHeight / 48)
    console.log('xSize', this.xSize)
    console.log('ySize', this.ySize)
    this.calculationRoomList(0, this.ySize)
    this.calculationInitRoomStateList(0, 0, this.ySize, this.xSize, 1)
    // this.roomStateArray = this.roomState.filter(item => item.show)
    dome.onscroll = () => {
      this.isScroll = false
      this.scrollTop = -dome.scrollTop
      this.scrollLeft = -dome.scrollLeft
      const _left = parseInt(-this.scrollLeft / 96)
      const _top = parseInt(-this.scrollTop / 48)
      this.calculationRoomList(_top, this.ySize)
      this.calculationRoomStateList(_top, _left, this.ySize, this.xSize, 1)
      this.isScroll = false
    }
    this.globalResize(() => {
      this.xSize = Math.ceil(dome.clientWidth / 96)
      this.ySize = Math.ceil(dome.clientHeight / 48)
      const _left = parseInt(-this.scrollLeft / 96)
      const _top = parseInt(-this.scrollTop / 48)
      this.calculationRoomList(_top, this.ySize)
      this.calculationInitRoomStateList(_top, _left, this.ySize, this.xSize, 1)
    })
  }
}
</script>

<style lang="scss" scoped>
  @import "./../../../style/global-variable.scss";
  .u-box {
    display: flex;
    flex-direction: column;
    position: relative;
    width: 90%;
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
        width: 96 * 30px;
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
      flex: 1;

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
          background-image: url("./../../../assets/room-state/room-bg-01.png");

          .u-li {
            position: absolute;
            width: 100%;
            height: 48px;
            line-height: 48px;
            border-bottom: 1px solid #606266;
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
          background-image: url("./../../../assets/room-state/room-state-bg-01.png");

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
