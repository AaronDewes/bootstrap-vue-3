<template>
  <div ref="parent" :class="classes">
    <b-button
      :id="computedId"
      :variant="variant"
      :size="size"
      :class="[buttonClasses, split ? splitClass : toggleClass]"
      :disabled="disabled"
      :type="splitButtonType"
      v-bind="buttonAttr"
    >
      {{ text }}
      <slot name="button-content" />
    </b-button>
    <b-button
      v-if="split"
      :variant="splitVariant || variant"
      :size="size"
      :disabled="disabled"
      v-bind="splitAttr"
      :class="toggleClass"
      class="dropdown-toggle-split dropdown-toggle"
      data-bs-toggle="dropdown"
      aria-expanded="false"
    >
      <span class="visually-hidden">
        {{ toggleText }}
      </span>
    </b-button>
    <ul
      class="dropdown-menu"
      :class="[menuClass, dropdownMenuClasses]"
      :aria-labelledby="computedId"
      :role="role"
    >
      <slot />
    </ul>
  </div>
</template>

<script lang="ts">
import {Dropdown} from 'bootstrap'
import {computed, defineComponent, onMounted, PropType, ref} from 'vue'
import {ButtonVariant, Size} from '../types'
import mergeDeep from '../utils/mergeDeep'
import useId from '../composables/useId'
import useEventListener from '../composables/useEventListener'

export default defineComponent({
  name: 'BDropdown',
  props: {
    autoClose: {type: [Boolean, String], default: true},
    block: {type: Boolean, default: false},
    boundary: {type: [String, HTMLElement], default: 'clippingParents'},
    dark: {type: Boolean, default: false},
    disabled: {type: Boolean, default: false},
    dropup: {type: Boolean, default: false},
    dropright: {type: Boolean, default: false},
    dropleft: {type: Boolean, default: false},
    id: {type: String},
    menuClass: {type: [Array, Object, String]},
    noFlip: {type: Boolean, default: false},
    offset: {type: [Number, String], default: 0},
    popperOpts: {type: Object, default: () => ({})},
    right: {type: Boolean, default: false},
    role: {type: String, default: 'menu'},
    size: {type: String as PropType<Size>},
    split: {type: Boolean, default: false},
    splitButtonType: {type: String as PropType<'button' | 'submit' | 'reset'>, default: 'button'},
    splitClass: {type: [Array, Object, String]},
    splitHref: {type: String, default: null},
    noCaret: {type: Boolean, default: false},
    splitVariant: {type: String as PropType<ButtonVariant>},
    text: {type: String},
    toggleClass: {type: [Array, Object, String]},
    toggleText: {type: String, default: 'Toggle dropdown'},
    variant: {type: String as PropType<ButtonVariant>, default: 'secondary'},
  },
  emits: ['show', 'shown', 'hide', 'hidden'],
  setup(props, {emit}) {
    const parent = ref<HTMLElement>()
    const dropdown = ref<HTMLElement>()
    const instance = ref<Dropdown>()
    const computedId = useId(props.id, 'dropdown')

    useEventListener(parent, 'show.bs.dropdown', () => emit('show'))
    useEventListener(parent, 'shown.bs.dropdown', () => emit('shown'))
    useEventListener(parent, 'hide.bs.dropdown', () => emit('hide'))
    useEventListener(parent, 'hidden.bs.dropdown', () => emit('hidden'))

    const classes = computed(() => ({
      'btn-group': props.split,
      'dropdown': !props.split,
      'd-grid': props.block,
    }))

    const buttonClasses = computed(() => ({
      'dropdown-toggle': !props.split,
      'dropdown-toggle-no-caret': props.noCaret && !props.split,
    }))

    const dropdownMenuClasses = computed(() => ({
      'dropdown-menu-dark': props.dark,
    }))

    const buttonAttr = computed(() => ({
      'data-bs-toggle': props.split ? null : 'dropdown',
      'aria-expanded': props.split ? null : false,
      'ref': props.split ? null : dropdown,
      'href': props.split ? props.splitHref : null,
    }))

    const splitAttr = computed(() => ({
      ref: props.split ? dropdown : null,
    }))

    onMounted(() => {
      instance.value = new Dropdown(dropdown.value.$el, {
        autoClose: props.autoClose,
        boundary: props.boundary,
        offset: props.offset.toString(),
        reference: props.offset || props.split ? 'parent' : 'toggle',
        popperConfig(defaultConfig) {
          const dropDownConfig = {
            placement: 'bottom-start',
            modifiers: [],
          }

          if (props.dropup) {
            dropDownConfig.placement = props.right ? 'top-end' : 'top-start'
          } else if (props.dropright) {
            dropDownConfig.placement = 'right-start'
          } else if (props.dropleft) {
            dropDownConfig.placement = 'left-start'
          } else if (props.right) {
            dropDownConfig.placement = 'bottom-end'
          }

          if (props.noFlip) {
            dropDownConfig.modifiers.push({
              name: 'flip',
              options: {
                fallbackPlacements: [],
              },
            })
          }

          return mergeDeep(defaultConfig, mergeDeep(dropDownConfig, props.popperOpts))
        },
      })
    })

    return {
      parent,
      computedId,
      classes,
      buttonClasses,
      buttonAttr,
      splitAttr,
      dropdownMenuClasses,
      dropdown,
    }
  },
})
</script>
