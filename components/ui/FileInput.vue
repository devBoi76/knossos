<template>
  <label
    :class="{ 'long-style': longStyle }"
    @drop.prevent="handleDrop"
    @dragover.prevent
  >
    <slot></slot>
    {{ prompt }}
    <input
      type="file"
      :multiple="multiple"
      :accept="accept"
      @change="handleChange"
    />
  </label>
</template>

<script>
import { fileIsValid } from '~/plugins/fileUtils'

export default {
  name: 'FileInput',
  components: {},
  props: {
    prompt: {
      type: String,
      default: 'Select file',
    },
    multiple: {
      type: Boolean,
      default: false,
    },
    accept: {
      type: String,
      default: null,
    },
    /**
     * The max file size in bytes
     */
    maxSize: {
      type: Number,
      default: null,
    },
    showIcon: {
      type: Boolean,
      default: true,
    },
    shouldAlwaysReset: {
      type: Boolean,
      default: false,
    },
    longStyle: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      files: [],
    }
  },
  methods: {
    addFiles(files, shouldNotReset) {
      if (!shouldNotReset || this.shouldAlwaysReset) this.files = files

      const validationOptions = { maxSize: this.maxSize, alertOnInvalid: true }
      this.files = [...this.files].filter((file) =>
        fileIsValid(file, validationOptions)
      )

      if (this.files.length > 0) {
        this.$emit('change', this.files)
      }
    },
    handleDrop(e) {
      this.addFiles(e.dataTransfer.files)
    },
    handleChange(e) {
      this.addFiles(e.target.files)
    },
  },
}
</script>

<style lang="scss" scoped>
label {
  flex-direction: unset;
  max-height: unset;

  svg {
    height: 1rem;
  }

  input {
    display: none;
  }

  &.long-style {
    display: flex;
    padding: 1.5rem 2rem;
    justify-content: center;
    align-items: center;
    grid-gap: 0.5rem;
    background-color: var(--color-button-bg);
    border-radius: var(--size-rounded-sm);
    border: dashed 0.3rem var(--color-text);
    cursor: pointer;
    color: var(--color-text-dark);
  }
}
</style>
