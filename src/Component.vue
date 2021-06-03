<template>
  <div class="scroll-indicator">
    <template v-for="item in items">
      <slot name="indicator" :item="item">
        <div
          v-if="states[item.name]"
          :key="item.name"
          :class="'scroll-indicator__arrow scroll-indicator__arrow--' + item.name"
        >
          <svg
            aria-hidden="true"
            focusable="false"
            role="img"
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 448 512"
          >
            <path
              fill="currentColor"
              d="M311.03 131.515l-7.071 7.07c-4.686 4.686-4.686 12.284 0 16.971L387.887 239H12c-6.627 0-12 5.373-12 12v10c0 6.627 5.373 12 12 12h375.887l-83.928 83.444c-4.686 4.686-4.686 12.284 0 16.971l7.071 7.07c4.686 4.686 12.284 4.686 16.97 0l116.485-116c4.686-4.686 4.686-12.284 0-16.971L328 131.515c-4.686-4.687-12.284-4.687-16.97 0z"
            ></path>
          </svg>
        </div>
      </slot>
    </template>
  </div>
</template>

<script>
export default {
  props: {
    value: {
      type: Array,
      default: () => ['top', 'bottom'],
      //enum: ['top', 'right', 'left', 'bottom']
    },
    treshold: {
      type: Number,
      default: 50,
    },
  },
  data() {
    return {
      observer: null,
      states: {
        top: false,
        right: false,
        left: false,
        bottom: false,
      },
    }
  },
  computed: {
    items() {
      return [{ name: 'top' }, { name: 'right' }, { name: 'bottom' }, { name: 'left' }].filter(i =>
        this.value.includes(i.name),
      )
    },
  },
  methods: {
    calculate() {
      let target = this.$el.parentElement
      let sw = target.scrollWidth - target.clientWidth - this.treshold
      let sh = target.scrollHeight - target.clientHeight - this.treshold
      this.states.top = target.scrollTop >= this.treshold
      this.states.right = target.scrollLeft <= sw
      this.states.bottom = target.scrollTop <= sh
      this.states.left = target.scrollLeft >= this.treshold
    },
    attachListeners(el) {
      if (!el) return
      el.addEventListener('scroll', this.calculate)
      this.calculate(el)
      this.observer = new MutationObserver(() => this.calculate(el))
      this.observer.observe(el, { childList: true, subtree: true })
    },
    detachListeners(el) {
      if (!el) return
      el.removeEventListener('scroll', this.calculate)
      if (this.observer) this.observer.disconnect()
    },
  },
  mounted() {
    this.attachListeners(this.$el.parentElement)
  },
  beforeDestroy() {
    this.detachListeners(this.$el.parentElement)
  },
}
</script>

<style lang="scss">
.scroll-indicator {
  $height: 35px;
  $width: 50px;
  &__arrow {
    opacity: 1;
    color: #414141;
    border: 2px solid #e1e1e1;
    background-color: rgba(white, 0.9);
    display: flex;
    align-items: center;
    justify-content: center;
    width: $width;
    height: $height;
    position: absolute;
    pointer-events: none;
    z-index: 1;
    border-radius: 20px;
    animation: fadeIn 250ms ease-in-out;
    transform-origin: center;
    @keyframes fadeIn {
      from {
        opacity: 0;
      }
    }
    svg {
      width: 0.375em;
      font-size: 40px;
      @keyframes fadeInOut {
        0%,
        100% {
          opacity: 0.3;
        }
        50% {
          opacity: 1;
        }
      }
      animation: fadeInOut 2s ease-in-out infinite;
    }
    &--left {
      left: 0;
      top: 50%;
      transform: rotate(180deg) translate(0, -50%);
    }
    &--right {
      right: 0;
      top: 50%;
      transform: rotate(0deg) translate(0, -50%);
    }
    &--bottom {
      right: calc(50% - #{$width - $height});
      bottom: 0;
      transform: rotate(90deg) translate(0%, -50%);
    }
    &--top {
      right: calc(50% - #{$width - $height});
      top: 0;
      transform: rotate(-90deg) translate(0%, -50%);
    }
  }
}
</style>
