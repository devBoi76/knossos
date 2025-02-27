<template>
  <div>
    <div
      :class="{
        shown: shown,
        noblur: !$orElse($cosmetics.advancedRendering, true),
      }"
      class="modal-overlay"
      @click="hide"
    />
    <div class="modal-body" :class="{ shown: shown }">
      <template v-if="shown">
        <div v-if="header" class="header">
          <h1>{{ header }}</h1>
          <button class="iconified-button icon-only transparent" @click="hide">
            <CrossIcon />
          </button>
        </div>
        <div class="content">
          <slot></slot>
        </div>
      </template>
    </div>
  </div>
</template>

<script>
import CrossIcon from '~/assets/images/utils/x.svg?inline'

export default {
  name: 'Modal',
  components: {
    CrossIcon,
  },
  props: {
    header: {
      type: String,
      default: null,
    },
  },
  data() {
    return {
      shown: false,
    }
  },
  methods: {
    show() {
      this.shown = true
    },
    hide() {
      this.shown = false
    },
  },
}
</script>

<style lang="scss" scoped>
.modal-overlay {
  visibility: hidden;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 20;

  transition: all 0.3s ease-in-out;

  &.shown {
    opacity: 1;
    visibility: visible;
    background: hsla(0, 0%, 0%, 0.5);
    backdrop-filter: blur(3px);
  }

  &.noblur {
    backdrop-filter: none;
  }
}

.modal-body {
  position: fixed;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 21;
  box-shadow: var(--shadow-raised), var(--shadow-inset);
  border-radius: var(--size-rounded-lg);
  max-height: calc(100% - 2 * var(--spacing-card-bg));
  overflow-y: auto;
  width: 600px;

  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: var(--color-bg);
    padding: var(--spacing-card-md) var(--spacing-card-lg);

    h1 {
      font-size: 1.25rem;
    }
  }

  .content {
    background-color: var(--color-raised-bg);
  }

  top: calc(100% + 400px);
  visibility: hidden;
  opacity: 0;
  transition: all 0.25s ease-in-out;

  &.shown {
    opacity: 1;
    visibility: visible;
    top: 50%;
  }

  @media screen and (max-width: 650px) {
    width: calc(100% - 2 * var(--spacing-card-bg));
  }
}
</style>
