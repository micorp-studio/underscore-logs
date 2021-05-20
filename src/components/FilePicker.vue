<template>
  <span
    class="
      relative
      inline-flex
      justify-between
      items-center
      bg-white bg-opacity-10
      hover:bg-opacity-20
      text-white text-opacity-80
      overflow-hidden
      rounded-md
      py-1
      px-3
      backdrop-filter backdrop-blur
    "
  >
    <span class="px-1"><slot></slot></span>
    <svg
      xmlns="http://www.w3.org/2000/svg"
      class="h-5 w-5"
      fill="none"
      viewBox="0 0 24 24"
      stroke="currentColor"
    >
      <path
        stroke-linecap="round"
        stroke-linejoin="round"
        stroke-width="2"
        d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12"
      />
    </svg>
    <input
      type="file"
      class="absolute opacity-0 cursor-pointer top-0 right-0 bottom-0 block"
      v-bind="$attrs"
    />
  </span>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  setup() {},
})
</script>

<style></style>
