<template>
  <label class="du-form-control w-full">
    <!-- Top labeles container -->
    <div v-if="showTopLabel" class="du-label relative">
      <!-- Left label -->
      <span v-if="topLeftLabel" class="du-label-text absolute left-0">
        {{ topLeftLabel }}
      </span>
      <!-- Right lebel -->
      <span v-if="topRightLabel" class="du-label-text-alt absolute right-0">
        {{ topRightLabel }}
      </span>
    </div>
    <!-- Input container -->
    <div class="relative">
      <!-- Input -->
      <input
        v-model="model"
        v-bind="validateAttrs"
        :type="modelType"
        :placeholder="placeholder"
        class="du-input du-input-bordered my-2 w-full max-w-full"
        :class="{
          ...inputSizeClass
        }"
      />
      <!-- Buttons container -->
      <div class="button-container">
        <div class="button-shell">
          <!-- Password type toggler -->
          <template v-if="inputType.password">
            <!-- Show password button -->
            <button v-if="passwordType.password" class="flex">
              <Icon
                name="mdi:eye-off"
                class="text-main-2 dark:text-white"
                :size="iconSize"
                @click="modelType = BaseInputType.text"
              />
            </button>
            <!-- Hide password button -->
            <button v-else class="flex">
              <Icon
                name="mdi:eye"
                class="text-main-2 dark:text-white"
                :size="iconSize"
                @click="modelType = BaseInputType.password"
              />
            </button>
          </template>
          <!-- Clear button -->
          <button v-if="showCloseButton" class="flex" @click="clearModel">
            <Icon
              name="vaadin:close"
              class="text-main-2 dark:text-white"
              :size="iconSize"
            />
          </button>
        </div>
      </div>
    </div>
    <!-- Bottom labeles -->
    <div v-if="showBottomLabel" class="du-label relative">
      <!-- Left label -->
      <span v-if="bottomLeftLabel" class="du-label-text-alt absolute left-0">
        {{ bottomLeftLabel }}
      </span>
      <!-- Right label -->
      <span v-if="bottomRightLabel" class="du-label-text-alt absolute right-0">
        {{ bottomRightLabel }}
      </span>
    </div>
    <div v-if="showError as boolean" class="flex text-xs text-red">
      {{ errors[name] }}
    </div>
  </label>
</template>

<script setup lang="ts">
import { BaseInputSize, BaseInputType, EMPTY_STRING } from '@/constants/global'
import { toTypedSchema } from '@vee-validate/zod'
import { useForm } from 'vee-validate'
import { z } from 'zod'
import type { BaseInputProps } from './types/BaseInputProps'

const emit = defineEmits(['update:modelValue'])
const props = withDefaults(defineProps<BaseInputProps>(), {
  modelValue: EMPTY_STRING,
  placeholder: EMPTY_STRING,
  size: BaseInputSize.md,
  type: BaseInputType.text
})

// VeeValidate
const { errors, defineField } = useForm({
  validationSchema: toTypedSchema(
    z.object({
      [props.name]: props.validate?.rules ?? z.any()
    })
  )
})

const [validateValue, validateAttrs] = defineField(props.name, {
  validateOnModelUpdate: props.validate?.validateOnUpdate ?? true
})
//

const modelType = ref<BaseInputType>(props.type)

const model = computed({
  get: () => props.modelValue,
  set: (value: string) => {
    validateValue.value = value
    emit('update:modelValue', value)
  }
})
const showError = computed(
  () => Boolean(props.validate) && Boolean(errors.value[props.name])
)
const showTopLabel = computed(() => props.topLeftLabel ?? props.topRightLabel)
const showBottomLabel = computed(
  () => props.bottomLeftLabel ?? props.bottomRightLabel
)
const showCloseButton = computed(() => model.value && props.clearButton)
const inputSizeClass = computed(() => ({
  'du-input-xs': inputSize.value.xs,
  'du-input-sm': inputSize.value.sm,
  'du-input-md': inputSize.value.md,
  'du-input-lg': inputSize.value.lg
}))
const inputType = computed(() => ({
  password: props.type === BaseInputType.password,
  text: props.type === BaseInputType.text
}))
const inputSize = computed(() => ({
  xs: props.size === BaseInputSize.xs,
  sm: props.size === BaseInputSize.sm,
  md: props.size === BaseInputSize.md,
  lg: props.size === BaseInputSize.lg
}))
const iconSize = computed(() => {
  switch (true) {
    case inputSize.value.xs:
      return '8'
    case inputSize.value.sm:
      return '12'
    case inputSize.value.md:
      return '16'
    case inputSize.value.lg:
      return '20'
  }
})
const passwordType = computed(() => ({
  password: modelType.value === BaseInputType.password,
  text: modelType.value === BaseInputType.text
}))

const clearModel = () => emit('update:modelValue', EMPTY_STRING)
</script>

<style lang="scss" scoped>
.button-container {
  @apply absolute right-3 top-0 flex h-full items-center justify-center gap-2 outline-none;
}

.button-shell {
  @apply flex items-center gap-2 bg-white pl-2 dark:bg-main-2;
}
</style>
