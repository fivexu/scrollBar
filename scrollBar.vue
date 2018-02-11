<template>
  <div id="scroll-bar"
       ref="scrollBar"
       @mouseenter.prev="scrollBarMouseEnter"
       @mouseleave="scrollBarMouseLeave"
       @mouseup="barMouseUp">
    <div class="content"
         ref="content"
         @touchstart="contentTouchStart"
         @touchmove="contentTouchMove"
         @touchend="contentTouchEnd">
      <slot></slot>
    </div>
    <div class="bar"
         :class="{ac:barShow,hide:barHide,barClass:true}"
         ref="bar"
         @mousedown="barMouseDown"
         @mousemove="barMouseMove"
         @mouseup="barMouseUp"
    ></div>
  </div>
</template>

<script type="text/ecmascript-6">
  export default {
    props: {
      barHide: {
        type: Boolean,
        default: false
      },
      barClass: {
        type: String,
        default: ''
      }
    },
    data() {
      return {
        barShow: false
      }
    },
    created() {
      this.touch = {}
    },
    methods: {
      setBarHeight() {
        let disHeight = this.$refs.content.children[0].offsetHeight - this.$refs.scrollBar.offsetHeight
        let barHeight = this.$refs.scrollBar.offsetHeight - disHeight
        if (disHeight >= 0) {
          if (barHeight <= 20) {
            barHeight = 20
          }
          this.$refs.bar.style.height = `${barHeight}px`
        }
      },
      addMousewheel(fn, obj = document) {
        if ((navigator.userAgent.toLowerCase().indexOf('firefox') !== -1)) {
          obj.addEventListener('DOMMouseScroll', fn, false)
        } else if (obj.addEventListener) {
          obj.addEventListener('mousewheel', fn, false)
        } else if (obj.attachEvent) {
          obj.attachEvent('onmousewheel', fn)
        } else {
          obj.onmousewheel = fn
        }
      },
      scrollBarMouseEnter() {
        clearTimeout(this.MousewheelTimer)
        if (!this.$refs.scrollBar || this.$refs.content.children[0].offsetHeight <= this.$refs.scrollBar.offsetHeight) {
          return
        }
        this.barShow = true
        let top = -this.$refs.content.offsetTop
        let percent = (this.$refs.scrollBar.offsetHeight - this.$refs.bar.offsetHeight) / (this.$refs.content.children[0].offsetHeight - this.$refs.scrollBar.offsetHeight)
        this.addMousewheel((ev) => {
          ev = ev || event
          ev.preventDefault()
          let delta = ev.wheelDelta || -ev.detail
          if (delta < 0) {
            top += 10
            if (top >= this.$refs.content.children[0].offsetHeight - this.$refs.scrollBar.offsetHeight) {
              top = this.$refs.content.children[0].offsetHeight - this.$refs.scrollBar.offsetHeight
            }
          }
          if (delta > 0) {
            top -= 10
            if (top <= 0) {
              top = 0
            }
          }
          this.$refs.bar.style.top = `${percent * top}px`
          this.$refs.content.style.top = `${-top}px`
        }, this.$refs.scrollBar)
      },
      scrollBarMouseLeave() {
        clearTimeout(this.MousewheelTimer)
        this.MousewheelTimer = setTimeout(() => {
          this.barShow = false
        }, 1000)
      },
      barMouseDown(ev) {
        ev = ev || event
        if (!this.$refs.scrollBar || this.$refs.content.children[0].offsetHeight <= this.$refs.scrollBar.offsetHeight) {
          return
        }
        this.mouseDowned = true
        this.start = ev.clientY - this.$refs.scrollBar.offsetTop - this.$refs.bar.offsetTop
      },
      barMouseMove(ev) {
        ev = ev || event
        if (!this.$refs.scrollBar || this.$refs.content.children[0].offsetHeight <= this.$refs.scrollBar.offsetHeight) {
          return
        }
        if (!this.mouseDowned) {
          return
        }
        let scrollBarTop = this.$refs.scrollBar.offsetTop
        let scrollBarHeight = this.$refs.scrollBar.offsetHeight
        let contentHeight = this.$refs.content.children[0].offsetHeight
        let barHeight = this.$refs.bar.offsetHeight
        let top = ev.clientY - scrollBarTop - this.start
        let percent = (contentHeight - scrollBarHeight) / (scrollBarHeight - barHeight)
        let contentTop = percent * top
        if (top <= 0) {
          top = 0
        }
        if (top >= this.$refs.scrollBar.offsetHeight - this.$refs.bar.offsetHeight) {
          top = this.$refs.scrollBar.offsetHeight - this.$refs.bar.offsetHeight
        }
        if (contentTop <= 0) {
          contentTop = 0
        }
        if (contentTop >= contentHeight - scrollBarHeight) {
          contentTop = contentHeight - scrollBarHeight
        }
        this.$refs.bar.style.top = `${top}px`
        this.$refs.content.style.top = `-${contentTop}px`
      },
      barMouseUp(ev) {
        ev = ev || event
        this.mouseDowned = false
      },
      contentTouchStart(ev) {
        ev = ev || event
        ev.preventDefault()
        if (!this.$refs.scrollBar || this.$refs.content.children[0].offsetHeight <= this.$refs.scrollBar.offsetHeight) {
          return
        }
        clearTimeout(this.touchTimer)
        this.barShow = true
        this.touch.initiated = true
        const touch = ev.touches[0]
        this.deltaY = 0
        this.touch.startY = touch.pageY - this.$refs.content.offsetTop
      },
      contentTouchMove(ev) {
        ev = ev || event
        if (!this.$refs.scrollBar || this.$refs.content.children[0].offsetHeight <= this.$refs.scrollBar.offsetHeight) {
          return
        }
        clearTimeout(this.touchTimer)
        if (!this.touch.initiated) {
          return
        }
        const touch = ev.touches[0]
        let percent = (this.$refs.scrollBar.offsetHeight - this.$refs.bar.offsetHeight) / (this.$refs.content.children[0].offsetHeight - this.$refs.scrollBar.offsetHeight)
        this.deltaY = -(touch.pageY - this.touch.startY)
        if (this.deltaY <= 0) {
          this.deltaY = 0
        }
        if (this.deltaY >= this.$refs.content.children[0].offsetHeight - this.$refs.scrollBar.offsetHeight) {
          this.deltaY = this.$refs.content.children[0].offsetHeight - this.$refs.scrollBar.offsetHeight
        }
        this.$refs.content.style.top = `${-this.deltaY}px`
        this.$refs.bar.style.top = `${this.deltaY * percent}px`
      },
      contentTouchEnd() {
        clearTimeout(this.touchTimer)
        this.touchTimer = setTimeout(() => {
          this.barShow = false
        }, 1000)
      }
    },
    mounted() {
      if (!this.$refs.scrollBar) {
        return
      }
      clearInterval(this.setTimer)
      this.setTimer = setInterval(() => {
        this.setBarHeight()
        setTimeout(() => {
          clearInterval(this.setTimer)
        }, 3000)
        if (this.$refs.bar.offsetHeight >= 20) {
          clearInterval(this.setTimer)
        }
      }, 30)
      document.body.addEventListener('mouseup', () => {
        this.mouseDowned = false
      })
    }
  }
</script>

<style lang="less" rel="stylesheet/less">
  @keyframes show {
    0% {
      transform: translate3d(100%, 0, 0);
      opacity: 0;
    }
    100% {
      transform: translate3d(0, 0, 0);
      opacity: 1;
    }
  }

  #scroll-bar {
    user-select: none;
    width: 100%;
    height: 100%;
    position: relative;
    overflow: hidden;
    .content {
      width: 100%;
      height: 100%;
      position: absolute;
      left: 0;
      top: 0;
    }
  }

  .bar {
    cursor: default;
    position: absolute;
    transform: translate3d(100%, 0, 0);
    right: 0;
    top: 0;
    width: 5px;
    height: 10px;
    border-radius: 2.5px;
    background-color: rgba(0, 0, 0, 0.3);
    &.ac {
      animation: show 0.3s;
      animation-fill-mode: forwards;
    }
    &.hide {
      display: none;
    }
  }
</style>