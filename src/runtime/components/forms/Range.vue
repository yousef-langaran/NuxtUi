<template>
  <div :class="wrapperClass">
    <input
        v-if="range"
        :id="inputId"
        ref="input"
        v-model.number="startValue"
        :name="name"
        :min="min"
        :max="max"
        :disabled="disabled"
        :step="step"
        type="range"
        :class="[inputClass, thumbClass, trackClass]"
        v-bind="attrs"
        @change="onChange"
    >
    <input
        :id="inputId"
        ref="input"
        v-model.number="endValue"
        :name="name"
        :min="min"
        :max="max"
        :disabled="disabled"
        :step="step"
        type="range"
        :class="[inputClass, thumbClass, trackClass]"
        v-bind="attrs"
        @change="onChange"
    >
    <span :class="progressClass" :style="progressStyle"/>
  </div>
</template>

<script lang="ts">
import {computed, toRef, defineComponent} from 'vue'
import type {PropType} from 'vue'
import {twMerge, twJoin} from 'tailwind-merge'
import {useUI} from '../../composables/useUI'
import {useFormGroup} from '../../composables/useFormGroup'
import {mergeConfig} from '../../utils'
import type {RangeSize, RangeColor, Strategy} from '../../types'
// @ts-expect-error
import appConfig from '#build/app.config'
import {range} from '#ui/ui.config'

const config = mergeConfig<typeof range>(appConfig.ui.strategy, appConfig.ui.range, range)

export default defineComponent({
  inheritAttrs: false,
  props: {
    modelValue: {
      type: Number,
      default: 0
    },
    id: {
      type: String,
      default: null
    },
    name: {
      type: String,
      default: null
    },
    disabled: {
      type: Boolean,
      default: false
    },
    min: {
      type: Number,
      default: 0
    },
    max: {
      type: Number,
      default: 100
    },
    step: {
      type: Number,
      default: 1
    },
    size: {
      type: String as PropType<RangeSize>,
      default: null,
      validator(value: string) {
        return Object.keys(config.size).includes(value)
      }
    },
    color: {
      type: String as PropType<RangeColor>,
      default: () => config.default.color,
      validator(value: string) {
        return appConfig.ui.colors.includes(value)
      }
    },
    inputClass: {
      type: String,
      default: null
    },
    class: {
      type: [String, Object, Array] as PropType<any>,
      default: () => ''
    },
    range:{
      type: Boolean,
      default: false
    },
    ui: {
      type: Object as PropType<Partial<typeof config> & { strategy?: Strategy }>,
      default: () => ({})
    }
  },
  emits: ['update:modelValue', 'change'],
  setup(props, {emit}) {
    const {ui, attrs} = useUI('range', toRef(props, 'ui'), config)

    const {emitFormChange, inputId, color, size, name} = useFormGroup(props, config)

    const value = computed({
      get() {
        return props.modelValue
      },
      set(value) {
        emit('update:modelValue', value)
      }
    })

    const onChange = (event: Event) => {
      emit('change', (event.target as HTMLInputElement).value)
      emitFormChange()
    }

    const wrapperClass = computed(() => {
      return twMerge(twJoin(
          ui.value.wrapper,
          ui.value.size[size.value]
      ), props.class)
    })

    const inputClass = computed(() => {
      return twMerge(twJoin(
          ui.value.base,
          ui.value.background,
          ui.value.rounded,
          color.value && ui.value.ring.replaceAll('{color}', color.value),
          ui.value.size[size.value]
      ), props.inputClass)
    })

    const thumbClass = computed(() => {
      return twJoin(
          ui.value.thumb.base,
          // Intermediate class to allow thumb ring or background color (set to `current`) as it's impossible to safelist with arbitrary values
          color.value && ui.value.thumb.color.replaceAll('{color}', color.value),
          ui.value.thumb.ring,
          ui.value.thumb.background,
          ui.value.thumb.size[size.value]
      )
    })

    const trackClass = computed(() => {
      return twJoin(
          ui.value.track.base,
          ui.value.track.background,
          ui.value.track.rounded,
          ui.value.track.size[size.value]
      )
    })

    const progressClass = computed(() => {
      return twJoin(
          ui.value.progress.base,
          ui.value.progress.rounded,
          color.value && ui.value.progress.background.replaceAll('{color}', color.value),
          ui.value.progress.size[size.value]
      )
    })
    const progressStyle = computed(() => {
      const { min, max } = props
      const clampedStart = props.range ? Math.max(min, Math.min(startValue.value, max)) : 0
      const clampedEnd = Math.max(min, Math.min(endValue.value, max))
      const relativeStart = props.range ? (clampedStart - min) / (max - min) : 0
      const relativeEnd = (clampedEnd - min) / (max - min)
      return {
        left: `${relativeStart * 100}%`,
        width: `${(relativeEnd - relativeStart) * 100}%`
      }
    })
    const updateValue = (value) => {
      if (value[0] > value[1] && props.range) {
        value = [value[1], value[0]]
      }
      emit('update:modelValue', value)
    }
    const startValue = computed({
      get() {
        return props.range ? props.modelValue[0] : 0
      },
      set(value) {
        updateValue([value, props.modelValue[1]])
      }
    })
    const endValue = computed({
      get() {
        return props.range ? props.modelValue[1] : props.modelValue
      },
      set(value) {
        if (props.range){
          updateValue([props.modelValue[0], value])
        }
        else{
          updateValue(value)
        }
      }
    })

    return {
      startValue,
      endValue,
      // eslint-disable-next-line vue/no-dupe-keys
      ui,
      attrs,
      // eslint-disable-next-line vue/no-dupe-keys
      name,
      inputId,
      value,
      wrapperClass,
      // eslint-disable-next-line vue/no-dupe-keys
      inputClass,
      thumbClass,
      trackClass,
      progressClass,
      progressStyle,
      onChange
    }
  }
})
</script>
