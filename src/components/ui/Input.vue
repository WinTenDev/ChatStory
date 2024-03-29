<template>
  <div class="relative" :class="[block ? 'block' : 'inline-block']">
    <label v-if="label" class="text-gray-500 text-sm ml-2" :for="componentId">
      {{ label }}
      <slot name="label"></slot>
    </label>
    <div class="relative flex items-center">
      <span v-if="$slots.prepend" class="ml-3 left-0 absolute inline-block">
        <slot name="prepend"></slot>
      </span>
      <input
        v-bind="{ value: modelValue, type, placeholder, disabled, readonly, maxlength: max }"
        :id="componentId"
        class="w-full ui-input rounded-xl border border-gray-200 transition focus:ring focus:ring-opacity-50"
        :class="[
          error
            ? 'focus:ring-red-500 focus:border-red-500 border-red-500'
            : 'focus:border-primary focus:ring-primary',
          disabled ? 'bg-gray-100 text-gray-500' : 'bg-transparent',
          {
            'pr-10': $slots.append,
            'pl-10': $slots.prepend,
            'hide-appearance': hideAppearance,
          },
        ]"
        @input="emitValue"
        @blur="$emit('blur', $event)"
      />
      <span v-if="$slots.append" class="mr-3 right-0 absolute inline-block">
        <slot name="append"></slot>
      </span>
    </div>
    <div v-if="error || showDetail" class="text-sm ml-2 min-h-[1.5rem] flex items-start">
      <span
        v-if="error && errorMessage"
        class="inline-block leading-tight text-red-500 flex-1 text-overflow"
        :title="errorMessage"
      >
        {{ errorMessage }}
      </span>
      <span v-if="max">{{ modelValue.length }}/{{ max }}</span>
    </div>
  </div>
</template>
<script>
import { useComponentId } from '@/composable/componentId';

export default {
  props: {
    modelValue: {
      type: [Number, String],
      default: '',
    },
    label: {
      type: String,
      default: '',
    },
    type: {
      type: String,
      default: 'text',
    },
    errorMessage: {
      type: String,
      default: '',
    },
    placeholder: {
      type: String,
      default: '',
    },
    max: [Number, String],
    hideAppearance: Boolean,
    error: Boolean,
    block: Boolean,
    readonly: Boolean,
    disabled: Boolean,
    showDetail: Boolean,
  },
  emits: ['update:modelValue', 'input', 'change', 'blur'],
  setup(props, { emit }) {
    const componentId = useComponentId('input');

    function emitValue(event) {
      let { value } = event.target;
      const maxLength = Math.abs(props.max);

      if (value.length > maxLength) {
        value = value.slice(0, maxLength);
      }

      emit('update:modelValue', value);
      emit('input', value);
      emit('change', value);
    }

    return {
      emitValue,
      componentId,
    };
  },
};
</script>
<style>
.ui-input {
  text-align: inherit;
}
.ui-input.hide-appearance::-webkit-outer-spin-button,
.ui-input.hide-appearance::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
.ui-input.hide-appearance[type='number'] {
  -moz-appearance: textfield;
}
</style>
