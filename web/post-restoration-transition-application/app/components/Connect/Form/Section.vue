<script setup lang="ts">
defineProps<{
  title?: string
  error?: boolean
}>()
</script>

<template>
  <div
    data-testid="form-section"
    :class="[error ? 'border-l-2 border-red-600' : '']"
  >
    <div class="flex flex-col sm:flex-row">
      <div
        v-if="$slots.title || title"
        class="w-[200px] lg:w-[320px]"
      >
        <slot name="title" :error="error">
          <p class="mb-2 sm:mb-0 sm:font-bold" :class="error ? 'text-red-600' : ''">
            {{ title }}
          </p>
        </slot>
      </div>
      <div class="w-full sm:pl-5">
        <slot />
      </div>
    </div>
  </div>
</template>
