<template>
  <div class="veui-select" :ui="ui">
    <veui-button
      :ui="ui"
      :class="{'veui-button-empty': value === null}"
      :disabled="disabled || readonly"
      @click="expanded = !expanded"
      ref="veui-select-button">
      <slot name="label" :label="label">
        <span>{{ label }}</span>
      </slot>
      <icon :name="`caret-${expanded ? 'up' : 'down'}`"></icon>
    </veui-button>
    <veui-overlay
      v-if="expanded"
      overlay-class="veui-select-overlay"
      target="veui-select-button"
      :open="expanded"
      :options="overlay">
      <div class="veui-select-options" v-clickoutside="close">
        <slot>
          <template v-for="option in options">
            <veui-option
              v-if="option.value"
              v-bind="option"
              :selected="option.value === value"
              :key="option.value"
              :icon="optionicon"
              @select="handleSelect">
                <slot name="option" v-bind="option"></slot>
            </veui-option>
            <div v-else-if="option.options" class="veui-select-option-group">
              <slot name="option-label" :label="option.label">
                <div class="veui-select-option-label">
                  <span>{{ option.label }}</span>
                </div>
              </slot>
              <veui-option
                v-for="subOption in option.options"
                v-bind="subOption"
                :selected="subOption.value === value"
                :key="subOption.value"
                :icon="optionicon"
                @select="handleSelect">
                <slot name="option" v-bind="subOption"></slot>
              </veui-option>
            </div>
          </template>
        </slot>
      </div>
    </veui-overlay>
  </div>
</template>

<script>
import Icon from '../Icon'
import Button from '../Button'
import Option from './Option'
import Overlay from '../Overlay'
import mixin from '../../mixins/input'
import 'vue-awesome/icons/caret-down'
import 'vue-awesome/icons/caret-up'

export default {
  name: 'veui-select',
  mixins: [mixin],
  model: {
    event: 'change'
  },
  components: {
    'icon': Icon,
    'veui-button': Button,
    'veui-option': Option,
    'veui-overlay': Overlay
  },
  props: {
    ui: String,
    placeholder: {
      type: String,
      default: '请选择'
    },
    multiple: {
      type: Boolean,
      default: false
    },
    optionicon: {
      type: Boolean,
      default: false
    },
    options: Array
  },
  data () {
    return {
      expanded: false,
      overlay: {
        attachment: 'top left',
        targetAttachment: 'bottom left',
        offset: '-3px 0',
        constraints: [
          {
            to: 'scrollParent',
            attachment: 'together'
          }
        ]
      }
    }
  },
  computed: {
    labelMap () {
      let mapOptions = {}
      function extract (options) {
        if (!options) {
          return
        }
        options.forEach(option => {
          if (option.value) {
            mapOptions[option.value] = option.label
          } else if (option.options) {
            extract(option.options)
          }
        })
      }
      extract(this.options)
      return mapOptions
    },
    label () {
      if (this.value === null) {
        return this.placeholder
      }
      return this.labelMap[this.value]
    }
  },
  methods: {
    handleSelect (val) {
      this.expanded = false
      this.$emit('change', val)
    },
    close () {
      // FIXME: prevent being reversed by button click for now
      setTimeout(() => {
        this.expanded = false
      })
    }
  }
}
</script>

<style lang="less">
@import "../../styles/theme-default/lib.less";
.veui-select {
  display: inline-block;
  width: 160px;

  .veui-button {
    width: 100%;
    position: relative;
    text-align: left;
    span {
      display: inline-block;
      max-width: ~"calc(100% - 16px)";
      width: 100%;
      .ellipsis();
    }
    &.veui-button-empty {
      color: @veui-gray-color-weak;
    }
    .veui-icon {
      float: right;
    }
  }
  &-options {
    min-width: 160px;
    max-height: 280px;
    overflow-y: auto;
    background-color: #fff;
    border: 1px solid @veui-gray-color-sup-2;
    border-radius: 2px;
  }

  &-option-group {
    .veui-select-option-label {
      display: inline-block;
      .centered-line(@veui-height-normal);
      padding: 0 10px;
      font-size: @veui-font-size-small;
      color: @veui-gray-color-weak;
    }

    .veui-option {
      padding-left: 20px;
    }
  }
}
</style>