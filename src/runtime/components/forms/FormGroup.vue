<template>
  <div :class="ui.wrapper" v-bind="attrs">
    <div v-if="label || $slots.label" :class="[ui.label.wrapper, size]">
      <label :for="inputId" :class="[ui.label.base, required ? ui.label.required : '']">
        <slot v-if="$slots.label" name="label" v-bind="{ error, label, name, hint, description, help }" />
        <template v-else>{{ label }}</template>
      </label>
      <span v-if="hint || $slots.hint" :class="[ui.hint]">
        <slot v-if="$slots.hint" name="hint" v-bind="{ error, label, name, hint, description, help }" />
        <template v-else>{{ hint }}</template>
      </span>
    </div>

    <p v-if="description || $slots.description" :class="[ui.description, size]">
      <slot v-if="$slots.description" name="description" v-bind="{ error, label, name, hint, description, help }" />
      <template v-else>
        {{ description }}
      </template>
    </p>

    <div :class="[label ? ui.container : '']">
      <slot v-bind="{ error }" />

      <p v-if="(typeof error === 'string' && error) || $slots.error" :class="[ui.error, size]">
        <slot v-if="$slots.error" name="error" v-bind="{ error, label, name, hint, description, help }" />
        <template v-else>
          {{ error }}
        </template>
      </p>
      <p v-else-if="help || $slots.help" :class="[ui.help, size]">
        <slot v-if="$slots.help" name="help" v-bind="{ error, label, name, hint, description, help }" />
        <template v-else>
          {{ help }}
        </template>
      </p>
    </div>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, provide, inject, ref, toRef } from 'vue'
import type { Ref, PropType } from 'vue'
import { useUI } from '../../composables/useUI'
import { mergeConfig } from '../../utils'
import type { FormError, InjectedFormGroupValue, FormGroupSize, Strategy } from '../../types'
// @ts-expect-error
import appConfig from '#build/app.config'
import { formGroup } from '#ui/ui.config'

const config = mergeConfig<typeof formGroup>(appConfig.ui.strategy, appConfig.ui.formGroup, formGroup)

export default defineComponent({
  inheritAttrs: false,
  props: {
    name: {
      type: String,
      default: null
    },
    size: {
      type: String as PropType<FormGroupSize>,
      default: null,
      validator (value: string) {
        return Object.keys(config.size).includes(value)
      }
    },
    label: {
      type: String,
      default: null
    },
    description: {
      type: String,
      default: null
    },
    required: {
      type: Boolean,
      default: false
    },
    help: {
      type: String,
      default: null
    },
    error: {
      type: [String, Boolean],
      default: null
    },
    hint: {
      type: String,
      default: null
    },
    class: {
      type: [String, Object, Array] as PropType<any>,
      default: undefined
    },
    ui: {
      type: Object as PropType<Partial<typeof config & { strategy?: Strategy }>>,
      default: undefined
    },
    eagerValidation: {
      type: Boolean,
      default: false
    }
  },
  setup (props) {
    const { ui, attrs } = useUI('formGroup', toRef(props, 'ui'), config, toRef(props, 'class'))

    const formErrors = inject<Ref<FormError[]> | null>('form-errors', null)

    const error = computed(() => {
      return (props.error && typeof props.error === 'string') || typeof props.error === 'boolean'
        ? props.error
        : formErrors?.value?.find((error) => error.path === props.name)?.message
    })

    const size = computed(() => ui.value.size[props.size ?? config.default.size])
    const inputId = ref()

    provide<InjectedFormGroupValue>('form-group', {
      error,
      inputId,
      name: computed(() => props.name),
      size: computed(() => props.size),
      eagerValidation: computed(() => props.eagerValidation)
    })

    return {
      // eslint-disable-next-line vue/no-dupe-keys
      ui,
      attrs,
      inputId,
      // eslint-disable-next-line vue/no-dupe-keys
      size,
      // eslint-disable-next-line vue/no-dupe-keys
      error
    }
  }
})
</script>
