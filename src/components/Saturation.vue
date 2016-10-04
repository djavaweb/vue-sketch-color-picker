<template lang="pug">
.sketch-color-picker--saturation(
:style="{background: bgColor}",
v-el:container,
@mousedown="handleMouseDown")
    .sketch-color-picker--saturation---white
    .sketch-color-picker--saturation---black
    .sketch-color-picker--saturation---pointer(:style="{top: pointerTop, left: pointerLeft}")
      slot
        .sketch-color-picker--saturation---circle
</template>

<script>
import throttle from 'lodash.throttle'
export default {
  name: 'Saturation',
  props: {
    colors: Object,
    onChange: Function
  },
  computed: {
    bgColor () {
      return 'hsl(' + this.colors.hsl.h + ',100%, 50%)'
    },
    pointerTop () {
      return -(this.colors.hsv.v * 100) + 100 + '%'
    },
    pointerLeft () {
      return this.colors.hsv.s * 100 + '%'
    }
  },
  methods: {
    throttle: throttle((fn, data) => {
      fn(data)
    }, 50),
    handleChange (e, skip) {
      !skip && e.preventDefault()
      var container = this.$els.container
      var containerWidth = container.clientWidth
      var containerHeight = container.clientHeight
      var left = (e.pageX || e.touches[0].pageX) - (container.getBoundingClientRect().left + window.pageXOffset)
      var top = (e.pageY || e.touches[0].pageY) - (container.getBoundingClientRect().top + window.pageYOffset)

      if (left < 0) {
        left = 0
      } else if (left > containerWidth) {
        left = containerWidth
      } else if (top < 0) {
        top = 0
      } else if (top > containerHeight) {
        top = containerHeight
      }

      var saturation = left * 100 / containerWidth
      var bright = -(top * 100 / containerHeight) + 100

      this.throttle(this.onChange, {
        h: this.colors.hsl.h,
        s: saturation,
        v: bright,
        a: this.colors.hsl.a,
        source: 'rgb'
      })
    },
    handleMouseDown (e) {
      this.handleChange(e, true)
      window.addEventListener('mousemove', this.handleChange)
      window.addEventListener('mouseup', this.handleMouseUp)
    },
    handleMouseUp (e) {
      this.unbindEventListeners()
    },
    unbindEventListeners () {
      window.removeEventListener('mousemove', this.handleChange)
      window.removeEventListener('mouseup', this.handleMouseUp)
    }
  }
}
</script>

<style lang="sass">  
.sketch-color-picker {
  &--saturation,
  &--saturation---white,
  &--saturation---black {
    cursor: pointer;
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
  }

  &--saturation---white {
    background: linear-gradient(to right, #fff, rgba(255,255,255,0));
  }

  &--saturation---black {
    background: linear-gradient(to top, #000, rgba(0,0,0,0));
  }

  &--saturation---pointer {
    cursor: pointer;
    position: absolute;
  }

  &--saturation---circle{
    cursor: head;
    width: 4px;
    height: 4px;
    box-shadow: 0 0 0 1.5px #fff, inset 0 0 1px 1px rgba(0,0,0,.3), 0 0 1px 2px rgba(0,0,0,.4);
    border-radius: 50%;
    transform: translate(-2px, -2px);
  }
}
</style>