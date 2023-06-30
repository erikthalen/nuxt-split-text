<script setup>
const props = defineProps({
  text: String,
  progress: [Number, String],
})

const textRef = ref(null)

const lines = ref([])
const children = ref(0)

// same as https://processing.org/reference/map_.html
// ex: reMap(1, 0, 5) -> 0.2
const remap = (value, low1, high1, low2 = 0, high2 = 1) => {
  return low2 + ((high2 - low2) * (value - low1)) / (high1 - low1)
}

const between = (progress, { easing, time: [min, max] }) => {
  // remaps min-max to 0-1
  const progressValue = unref(progress)
  if (min >= progressValue) return 0
  if (max <= progressValue) return 1

  return easing(remap(progressValue, min, max))
}

let accumulatedLineHeight = 0
let lastLinesMarginAndPadding = 0

watch(textRef, () => {
  if (!textRef.value) return

  children.value = [...textRef.value.children]

  children.value.forEach(child => {
    const { height } = child.getBoundingClientRect()
    const { lineHeight, marginTop, marginBottom, paddingTop, paddingBottom } =
      getComputedStyle(child)

    const childLineAmount = Math.round(height / parseFloat(lineHeight))

    const newLines = Array(childLineAmount)
      .fill(0)
      .map((_, idx) => {
        const firstChild = idx === 0
        const lastChild = idx === childLineAmount - 1

        // add any margin/paddings, including last line's margin/padding
        const extra = firstChild
          ? parseFloat(marginTop) +
            parseFloat(paddingTop) +
            lastLinesMarginAndPadding
          : 0

        // keep track of where this line starts
        accumulatedLineHeight += parseFloat(lineHeight) + extra

        // last line should inform next child's first line about its margin/padding
        lastLinesMarginAndPadding = lastChild
          ? parseFloat(marginBottom) + parseFloat(paddingBottom)
          : 0

        return {
          accumulatedLineHeight,
          lineHeight: parseFloat(lineHeight),
        }
      })

    lines.value = [...lines.value, ...newLines]
  })
})

const getStaggeredProgress = idx => {
  const section = 1 / lines.value.length
  const stagger = section * idx

  return between(props.progress, {
    easing: easeOutQuint,
    time: [stagger, stagger + section],
  })
}

const getMask = (line, idx) => {
  const start =
    line.accumulatedLineHeight - getStaggeredProgress(idx) * line.lineHeight
  const stop = line.accumulatedLineHeight

  const visible = 'black'
  const hidden = 'rgba(0,0,0,0)'

  return {
    maskImage: `linear-gradient(
      ${hidden} ${start}px,
      ${visible} ${start}px,
      ${visible} ${stop}px,
      ${hidden} ${stop}px)
    `,
  }
}

const getTransform = (line, idx) => {
  return {
    translate: `0 ${
      -line.lineHeight + getStaggeredProgress(idx) * line.lineHeight
    }px`,
  }
}
</script>

<template>
  <div class="container">
    <div
      class="real"
      ref="textRef"
      :style="{ opacity: progress !== '1' ? 0 : null }"
      v-html="text"
    />

    <div
      v-if="progress !== '0' && progress !== '1'"
      v-for="(line, idx) in lines"
      class="clone"
      :style="{
        ...getMask(line, idx),
        ...getTransform(line, idx),
      }"
      :key="line"
      v-html="text"
    />
  </div>
</template>

<style>
.container {
  position: relative;
}

.real,
.clone {
  display: flex;
  flex-direction: column;
}

.real > *,
.clone > * {
  display: inline-block;
}

:where(.real > *),
:where(.clone > *) {
  width: 100%;
}

.clone {
  position: absolute;
  top: 0;
  margin: 0;
}
</style>
