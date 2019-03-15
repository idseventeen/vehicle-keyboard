<template>
  <div class="vehicle-keyboard__wrap">
    <transition name="fade-in">
      <div class="vehicle-keyboard__mask" :style="maskStyle" v-show="showIn" @click="onMaskClick"></div>
    </transition>
    <transition name="slide-in">
      <div class="vehicle-keyboard" v-show="showIn">
        <!-- 预览区 -->
        <div class="vehicle-keyboard__view">
          <div class="vehicle-keyboard__energy" :class="{'checked': keyboardMode === 2}" @click="onToggleTypeClick">
            <svg xmlns="http://www.w3.org/2000/svg" class="icon toggle" viewBox="0 0 1024 1024">
              <path d="M575.915 725.333a21.397 21.397 0 0 1-21.248-21.162V319.829A21.184 21.184 0 0 1 576 298.667c11.776 0 21.333 9.706 21.333 21.162v333.91l85.568-85.568a21.227 21.227 0 0 1 30.102.064c8.32 8.32 8.213 21.973.085 30.101l-120.832 120.81a21.141 21.141 0 0 1-16.341 6.187zm-152.79-426.325a21.419 21.419 0 0 1 24.896 20.864v384.341a21.205 21.205 0 0 1-21.333 21.163c-11.797 0-21.355-9.707-21.355-21.163V364.267l-91.669 91.605a21.248 21.248 0 0 1-30.123-.064 21.419 21.419 0 0 1-.064-30.101l120.896-120.811a21.184 21.184 0 0 1 18.752-5.888zm252.203-181.29A425.43 425.43 0 0 0 512 85.332C276.352 85.333 85.333 276.352 85.333 512S276.352 938.667 512 938.667 938.667 747.648 938.667 512c0-56.747-11.094-112-32.384-163.328a21.333 21.333 0 0 0-39.403 16.341A382.763 382.763 0 0 1 896 512c0 212.075-171.925 384-384 384S128 724.075 128 512s171.925-384 384-384c51.115 0 100.8 9.984 146.987 29.12a21.333 21.333 0 0 0 16.341-39.403z"/>
            </svg>
            <span>新能源</span>
          </div>
          <div class="vehicle-keyboard__vehicle">
            <div class="vehicle-keyboard__num"
              :class="{ 'active': index === cursorIndex }"
              v-for="index in vehicleMaxLen" :key="index"
              @click="onInputKeyClick(index)"
            >{{vehicleNums[index - 1]}}</div>
          </div>
          <div class="vehicle-keyboard__done" @click="hide">完成</div>
        </div>

        <!-- 主键盘区 -->
        <div class="vehicle-keyboard__board vehicle-keyboard__board--label">
          <div class="vehicle-keyboard__row" v-for="(row, index) in keyboard[showKeyType]" :key="index">
            <div class="vehicle-keyboard__col"
              :class="{ 'disabled': disabledCodeKey(index, key) }"
              v-for="(key, idx) in row" :key="idx"
              @click="onKeyClick(index, key)">{{key}}</div>

            <div class="vehicle-keyboard__col delkey" :class="delKeyClasslist"
              v-if="index === keyboard[showKeyType].length - 1" @click="onDelClick">删除</div>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
const KEY_TYPE = {
  LABEL: 'label',
  CODE: 'code'
}

export default {
  name: 'VehicleKeyboard',
  model: {
    prop: 'showIn',
    event: 'update:showIn'
  },
  props: {
    vehicleKey: [String, Array],
    showIn: {
      type: Boolean,
      default: false
    },
    maskColor: String,
    hideOnMask: {
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      showKeyType: KEY_TYPE.LABEL, // 键盘类型
      keyboardMode: 1, // 1: 普通车牌 | 2: 新能源车牌
      keyboard: {
        label: [
          ['粤', '京', '冀', '沪', '津', '晋', '蒙', '辽', '吉', '黑'],
          ['苏', '浙', '皖', '闽', '赣', '鲁', '豫', '鄂', '湘', '桂'],
          ['琼', '渝', '川', '贵', '云', '藏', '陕', '甘', '青', '宁'],
          ['新', '临']
        ],
        code: [
          ['1', '2', '3', '4', '5', '6', '7', '8', '9', '0'],
          ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'J', 'K'],
          ['L', 'M', 'N', 'P', 'Q', 'R', 'S', 'T', 'U', 'V'],
          ['W', 'X', 'Y', 'Z', '港', '澳', '学']
        ]
      },

      vehicleNums: [],
      cursorIndex: 1 // 当前光标位置
    }
  },
  computed: {
    vehicleMaxLen () {
      return this.keyboardMode === 1 ? 7 : 8
    },
    delKeyClasslist () {
      return {
        'float-end': this.showKeyType === KEY_TYPE.LABEL,
        'disabled': this.vehicleNums.length === 0
      }
    },
    maskStyle () {
      return `background-color: ${this.maskColor || 'rgba(0,0,0,0.8)'}`
    }
  },
  watch: {
    vehicleKey (val) {
      this.mergeVehicleNum(val)
    },
    vehicleNums (val) {
      this.showKeyType = KEY_TYPE[val.length === 0 ? 'LABEL' : 'CODE']
    },
    showIn (val) {
      this.$emit('update:showIn', val)
      this.$emit(val ? 'on-show' : 'on-hide')
    }
  },
  methods: {
    // 禁用当前KEY
    disabledCodeKey (row, key) {
      const numKey = this.cursorIndex === 2 && row === 0 // 数字键
      const lastKey = this.cursorIndex < 7 && row === 3 && !/[W|X|Y|Z]/.test(key) // 末行中文键
      return this.showKeyType === KEY_TYPE.CODE && (numKey || lastKey)
    },
    // 合并车牌数据
    mergeVehicleNum (keys, len = this.vehicleMaxLen) {
      if (!keys) return
      if (!Array.isArray(keys)) {
        keys = keys.split('')
      }
      this.vehicleNums = keys.slice(0, len)
      this.cursorIndex = this.vehicleNums.length === len ? len : this.vehicleNums.length + 1
    },
    // 输入框KEY点击
    onInputKeyClick (index) {
      this.cursorIndex = +index
      this.showKeyType = KEY_TYPE[index === 1 ? 'LABEL' : 'CODE']
    },
    // 键盘点击处理
    onKeyClick (row, key) {
      const disabled = this.disabledCodeKey(row, key)
      if (disabled) return

      const len = this.vehicleNums.length
      const fillLen = this.cursorIndex - len

      if (len >= this.cursorIndex) {
        // replace
        this.vehicleNums.splice(this.cursorIndex - 1, 1, key)
      } else if (fillLen > 1) {
        // fill
        this.vehicleNums = this.vehicleNums.concat([...new Array(fillLen - 1), key])
      } else {
        // add
        this.vehicleNums.push(key)
      }

      if (this.cursorIndex < this.vehicleMaxLen) this.cursorIndex += 1
    },
    // 删除键点击处理
    onDelClick () {
      if (this.cursorIndex < this.vehicleNums.length) {
        this.vehicleNums.splice(this.cursorIndex - 1, 1, ' ')
      } else {
        this.vehicleNums.pop()
      }

      if (this.cursorIndex === 1) return
      this.cursorIndex -= 1
    },
    // 切换键盘类型
    onToggleTypeClick () {
      if (this.keyboardMode === 1) {
        if (this.cursorIndex === this.vehicleMaxLen && this.cursorIndex === this.vehicleNums.length) {
          this.cursorIndex += 1
        }
      } else {
        if (this.cursorIndex === this.vehicleMaxLen) {
          this.cursorIndex -= 1
          if (this.vehicleNums.length === this.vehicleMaxLen) {
            this.vehicleNums.pop()
          }
        }
      }

      this.keyboardMode = this.keyboardMode === 1 ? 2 : 1
    },
    hide () {
      this.$emit('ondone', { origin: this.vehicleNums, target: this.vehicleNums.join('') })
      this.$emit('update:showIn', false)
    },
    onMaskClick () {
      if (!this.hideOnMask) return
      this.hide()
    }
  },
  mounted () {
    this.mergeVehicleNum(this.vehicleKey)
  }
}
</script>

<style lang="scss" scoped>
$namespace: 'vehicle-keyboard';
$keyboard-zIndex: 5000;
$keyboard-distence: 6px;
$keyboard-font_size: 16px;
$keyboard-line_height: 2.4;
$keyboard-border-radius: 3px;

.#{$namespace}__wrap {
  position: relative;
}

.#{$namespace}__mask {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: $keyboard-zIndex;
}

.#{$namespace} {
  position: fixed;
  left: 0;
  bottom: 0;
  z-index: $keyboard-zIndex + 5;
  width: 100%;
  backface-visibility: hidden;
  font-size: $keyboard-font_size;
  line-height: $keyboard-line_height;
  text-align: center;

  > * {
    box-sizing: border-box;
  }
}

// -------------------
// 预览区
// -------------------
.#{$namespace}__view {
  $height: 44px;
  position: relative;
  display: flex;
  width: 100%;
  height: $height;
  padding: $keyboard-distence;
  background: #fff;
  align-items: center;
  font-size: 14px;

  &::before {
    $color: #e5e5e5;
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 1px;
    border-top: 1px solid $color;
    color: $color;
    transform-origin: 0 100%;
    transform: scaleY(0.5);
  }
}

.#{$namespace}__energy {
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;

  .toggle {
    $size: 18px;
    width: $size;
    height: $size;
    margin-right: $keyboard-distence / 3;

    path {
      fill: #333;
    }
  }

  &.checked {
    $color: #89c649;
    color: $color;

    .toggle path {
      fill: $color;
    }
  }
}

.#{$namespace}__vehicle {
  flex: 1;
  display: flex;
  height: 100%;
  margin: 0 6px;
}

.#{$namespace}__done {
  width: 34px;
  // opacity: 0.5;

  // &.done {
  color: #4479f5;
  opacity: 1;
  // }
}

.#{$namespace}__num {
  flex: 1;
  border: 1px solid #e5e5e5;
  border-radius: 4px;

  & + & {
    margin-left: 5px;
  }

  &.active {
    border-color: #ffc200;
  }
}

// -------------------
// 主键盘区
// -------------------

.#{$namespace}__board {
  width: 100%;
  padding: $keyboard-distence;
  background: #eee;
}

.#{$namespace}__row {
  display: flex;
  width: 100%;

  & + & {
    margin-top: $keyboard-distence;
  }
}

.#{$namespace}__col {
  $width: '(100% - #{$keyboard-distence} * 9) / 10';
  flex: 0 0 calc(#{$width});
  background: #fff;
  border-radius: $keyboard-border_radius;
  overflow: hidden;

  & + & {
    margin-left: $keyboard-distence;
  }

  // 最后一个按键
  &.delkey {
    $keyboard-distence-delkey: $keyboard-distence * 2;
    flex: 0 0 calc(#{$width} * 3 + #{$keyboard-distence-delkey});
    // background: #abb2bb;

    // 浮动右侧
    &.float-end {
      $keyboard-distence-delkey: $keyboard-distence * 6;
      margin-left: calc(#{$width} * 5 + #{$keyboard-distence-delkey});
    }
  }

  &:active {
    color: #333;
    background: #ccc;
  }

  &.disabled {
    background: #c3c8d0;
  }
}

// -------------------
// vue-transitions
// -------------------

.fade-in-enter-active,
.fade-in-leave-active {
  opacity: 1;
  transition: opacity linear 0.2s;
}

.fade-in-enter,
.fade-in-leave-to {
  opacity: 0;
}

.slide-in-enter-active {
  animation: slideIn ease 0.2s forwards;
}

.slide-in-leave-active {
  @extend .slide-in-enter-active;
  animation-direction: reverse;
}

@keyframes slideIn {
  from {
    transform: translate3d(0, 100%, 0);
  }

  to {
    transform: translate3d(0, 0, 0);
  }
}
</style>
