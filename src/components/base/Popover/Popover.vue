<template>
  <span class="popper-wrapper">
    <slot class="ref" name="reference" />
    <transition name="fade">
      <div
        v-show="!disabled && visible"
        ref="popper"
        role="popper"
        class="popper el-popover"
        :style="{width: width + 'px'}"
      >
        <h3 v-if="title" class="popper-title">{{ title }}</h3>
        <slot />
        <div v-if="arrow" class="popper-arrow" data-popper-arrow></div>
      </div>
    </transition>
  </span>
</template>
<script>
import { create } from './createPopper'
import './popover.styl'
import { on } from '../utils/dom'
import { sleep } from '../../../utils'
import { clickOutSideRow } from '@/utils/dom'

export default {
  props: {
    placement: {
      type: String,
      default: 'top'
    },
    modifiers: {
      type: Array,
      default: () => ([])
    },
    showEvents: {
      type: Array,
      default: () => (['click', 'focus'])
    },
    hideEvents: {
      type: Array,
      default: () => (['blur'])
    },
    tabindex: {
      type: Number,
      default: 0
    },
    closeOnClickModal: {
      type: Boolean,
      default: true
    },
    appendToBody: {
      type: Boolean,
      default: false
    },
    width: {
      type: Number,
      default: 300
    },
    title: {
      type: String,
      default: ''
    },
    disabled: {
      type: Boolean,
      default: false
    },
    arrow: {
      type: Boolean,
      default: true
    },
    openDelay: {
      type: Number,
      default: 0
    }
  },
  data() {
    return {
      popperInstance: null,
      modalAppendToBody: this.appendToBody,
      visible: false
    }
  },
  mounted() {
    const { placement, modifiers, showEvents, hideEvents, appendToBody } = this
    const { popper } = this.$refs
    let reference = this.$refs.reference
    reference = this.referenceElm = this.$slots.reference[0].elm
    if (this.arrow) {
      const arrow = this.arrow ? popper.querySelector('.popper-arrow') : null
      modifiers.push(
        {
          name: 'arrow',
          options: {
            element: arrow,
          },
        }
      )
    }

    const close = () => {
      this.visible = false
      this.$emit('hide')
      document.body.removeEventListener('click', ccc)
    }

    const ccc = clickOutSideRow(popper, close)

    const createPopper = async () => {
      const t = this.popperInstance = create(reference, popper, {
        placement,
        modifiers,
        showEvents,
        hideEvents,
      })()
      await this.$nextTick()
      await t.update()
    }

    showEvents.forEach(e => on(reference, e, async () => {
      if (this.visible) return
      this.$emit('show')
      this.visible = true
      await this.$nextTick()
      this.$nextTick()
      createPopper()
      await sleep(500)
      document.body.addEventListener('click', ccc)
    }))
    hideEvents.forEach(e => on(reference, e, () => {
      close()
    }))

    if (appendToBody) {
      document.body.appendChild(popper)
    }
  }
}
</script>

<style lang="stylus" scoped>
.popper {
  max-width: 100vw

  &-wrapper {
    width: 100%
    display: inline-block
  }

  &-title {
    margin-top: -10px
  }
}
</style>


