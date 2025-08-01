<script setup lang="ts">
import { computed } from 'vue'
import { bytesToHumanSize } from '~/utils/format'

const props = withDefaults(
  defineProps<{
    bytes?: number
    colorize?: boolean
    digits?: number
    percent?: boolean
    total?: number
    icon?: string
    percentRatio?: number
  }>(),
  {
    percent: true,
    colorize: true,
    digits: 2,
    percentRatio: 0.5,
  },
)

const KB = 1024
const MB = KB ** 2

const colorScale = [
  [80 * KB, 'color-scale-neutral'],
  [500 * KB, 'color-scale-low'],
  [1 * MB, 'color-scale-medium'],
  [10 * MB, 'color-scale-high'],
  [20 * MB, 'color-scale-critical'],
] as const

const color = computed(() => {
  if (!props.colorize)
    return colorScale[0]
  const bytes = props.bytes || 0
  for (const [limit, color] of colorScale) {
    if (bytes < limit)
      return color
  }
  return colorScale[colorScale.length - 1]?.[1]
})

const ratio = computed(() => props.total ? (props.bytes || 0) * 100 / props.total : 0)

const formatted = computed(() => bytesToHumanSize(props.bytes || 0, props.digits))
</script>

<template>
  <div v-if="bytes" :class="color" class="px-0.4em py-0.2em font-mono line-height-none bg-gray:5 dark:bg-gray:4 flex items-center">
    <div v-if="icon" :class="icon" class="mr-1" />
    {{ formatted[0] }}<span text-xs op75 ml-0.4>{{ formatted[1] }}</span>
    <slot name="after">
      <span v-if="percent && ratio > percentRatio" text-xs ml1 op-fade border="l base" pl1>{{ +(ratio.toFixed(1)) }}%</span>
    </slot>
  </div>
</template>
