<template>
  <div class="full-mask" v-if="props.show" @click.self.stop="changeShow">
    <div
      class="full-mask__poc"
      :style="{ left: props.location.left + 'px', top: props.location.top + 'px' }"
    >
      <div class="tool-opacity__child">
        <div class="slider">
          <label class="slider__label">透明度</label>
          <div class="slider__area" id="slider-area">
            <div class="slider__back" :style="`--value: ${props.value}%`"></div>
            <div
              class="slider__block"
              id="slider-block"
              :style="{ left: `calc(${props.value}% - 7.5px)` }"
            ></div>
          </div>
          <input class="slider__input" type="number" max="100" min="0" v-model="inputValue" />
          <span class="unit">%</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, watch, nextTick, computed } from 'vue'

interface Location {
  left: number
  top: number
}
const props = withDefaults(
  defineProps<{
    show: boolean
    value: number
    location: Location
  }>(),
  {
    show: false,
    value: 0,
    location: () => {
      return {
        left: 0,
        top: 0
      }
    }
  }
)
let startX = 0 // 鼠标按下的位置
const barWidth = 152 // 滑块的宽度
const emit = defineEmits(['update:show', 'update:value'])
const inputValue = computed({
  get: () => props.value,
  set: (value: number) => emit('update:value', value)
})
const mMove = (event: MouseEvent) => {
  event.stopPropagation()
  event.preventDefault()
  let disWidth = event.clientX - startX
  if (disWidth > barWidth) {
    disWidth = barWidth
  }
  if (disWidth < 0) {
    disWidth = 0
  }

  inputValue.value = Math.trunc((disWidth / barWidth) * 100)
}

function bindListener() {
  const box = document.getElementById('slider-area')
  box?.addEventListener('mousedown', (event: MouseEvent) => {
    event.preventDefault()
    event.stopPropagation()
    inputValue.value = Math.trunc(((event.clientX - startX) / barWidth) * 100)
    box?.addEventListener('mousemove', mMove)
  })
  box?.addEventListener('mouseup', () => {
    box?.removeEventListener('mousemove', mMove)
  })
}

watch([() => props.show], (val) => {
  if (val[0]) {
    nextTick(() => {
      startX = props.location.left - 132 - 1 + 70
      bindListener()
    })
  }
})

const changeShow = () => {
  emit('update:show', false)
}

onMounted(() => {
  inputValue.value = barWidth * (props.value / 100)
})
</script>
<style scoped lang="scss">
.full-mask {
  background: transparent;
  height: 100%;
  left: 0;
  position: fixed;
  top: 0;
  width: 100%;
  z-index: 99;
  .full-mask__poc {
    height: 0;
    position: absolute;
    width: 0;
    z-index: 2;
  }
  .tool-opacity__child {
    background: #fff;
    border-radius: 4px;
    box-shadow: 0 9px 28px 8px rgb(42 49 67 / 6%), 0 3px 6px -4px rgb(42 49 67 / 11%),
      0 6px 16px 0 rgb(42 49 67 / 8%);
    left: 18px;
    position: absolute;
    top: 44px;
    transform: translateX(-50%);
    padding: 4px 1px;
  }
  .slider {
    display: flex;
    align-items: center;
    background: #fff;
    box-sizing: border-box;
    cursor: default;
    height: 40px;
    padding: 0 20px 0 16px;
    z-index: 1;
    position: relative;
    .slider__label {
      color: #1b2337;
      width: 54px;
    }
    .slider__area {
      height: 40px;
      margin: 0 12px 0 0;
      position: relative;
      width: 152px;
      .slider__back {
        background: #e4e7ed;
        border-radius: 4px;
        cursor: pointer;
        height: 3px;
        position: absolute;
        top: 18px;
        width: 100%;
        // --value: 44%
        &::before {
          background: #0773fc;
          border-radius: 1.5px;
          content: '';
          height: 3px;
          left: 0;
          position: absolute;
          top: 0;
          width: var(--value);
        }
      }
      .slider__block {
        background: #fff;
        border: 1px solid #0773fc;
        border-radius: 50%;
        box-sizing: border-box;
        cursor: pointer;
        height: 15px;
        left: 0;
        // left: calc(44% - 7.5px);
        position: absolute;
        top: 12px;
        width: 15px;
      }
    }
    .slider__input {
      border: 1px solid rgba(0, 0, 0, 0.1);
      border-radius: 3px;
      box-sizing: border-box;
      color: #1b2337;
      height: 30px;
      line-height: 30px;
      text-align: center;
      width: 50px;
    }
    .unit {
      position: absolute;
      right: 5px;
      top: 10px;
    }
  }
}
</style>
