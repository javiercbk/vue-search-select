<template>
  <div class="ui fluid search selection dropdown"
       :class="{ 'active visible':showMenu, 'error': isError, 'disabled': isDisabled }"
       @click="openOptions"
       @focus="openOptions">
    <i class="dropdown" :class="dynamicClass"></i>
    <input class="search"
           autocomplete="off"
           tabindex="0"
           v-model="searchText"
           ref="input"
           @focus.prevent="openOptions"
           @keyup.esc="closeOptions"
           @blur="blurInput"
           @keydown.up="prevItem"
           @keydown.down="nextItem"
           @keydown.enter.prevent=""
           @keyup.enter.prevent="enterItem"
           @keydown.delete="deleteTextOrItem"/>
    <div class="text"
         :class="textClass" :data-vss-custom-attr="searchTextCustomAttr">{{inputText}}
    </div>
    <div class="menu"
         ref="menu"
         @mousedown.prevent
         :class="menuClass"
         :style="menuStyle"
         tabindex="-1">
      <template v-for="(option, idx) in options">
        <div class="item"
             :class="{ 'selected': option.selected, 'current': pointer === idx }"
             :data-vss-custom-attr="customAttrs[idx] ? customAttrs[idx] : ''"
             @click.stop="selectItem(option)"
             @mousedown="mousedownItem"
             @mouseenter="pointerSet(idx)">
          {{option.text}}
        </div>
      </template>
    </div>
  </div>
</template>

<script>
  /* event : select */
  import common from './common'
  import { baseMixin, commonMixin, optionAwareMixin } from './mixins'

  export default {
    mixins: [baseMixin, commonMixin, optionAwareMixin],
    props: {
      showMissingOptions: {
        type: Boolean,
        default: false
      },
      selectedOption: {
        type: Object,
        default: () => { return { value: '', text: '' } }
      },
      loadingIconClass: {
        type: String
      },
      httpClient: {
        type: Function,
        required: true
      },
      delayMillis: {
        type: Number,
        default: 500
      }
    },
    created () {
      this.originalValue = this.selectedOption
    },
    data () {
      return {
        showMenu: false,
        timeoutId: null,
        loading: false,
        searchText: '',
        originalValue: { text: '', value: '' },
        mousedownState: false, // mousedown on option menu
        pointer: 0
      }
    },
    computed: {
      optionsWithOriginal () {
        if (this.originalValue.value && this.showMissingOptions) {
          const hasOriginalValue = this.options.filter(o => o.value === this.originalValue).length === 1
          if (!hasOriginalValue) {
            return this.options.concat([this.originalValue])
          }
        }
        return this.options
      },
      searchTextCustomAttr () {
        if (this.selectedOption && this.selectedOption.value) {
          return this.customAttr(this.selectedOption)
        }
        return ''
      },
      inputText () {
        if (this.searchText) {
          return ''
        } else {
          let text = this.placeholder
          if (this.selectedOption.text) {
            text = this.selectedOption.text
          }
          return text
        }
      },
      customAttrs () {
        try {
          if (Array.isArray(this.optionsWithOriginal)) {
            return this.optionsWithOriginal.map(o => this.customAttr(o))
          }
        } catch (e) {
          // if there is an error, just return an empty array
        }
        return []
      },
      textClass () {
        if (!this.selectedOption.text && this.placeholder) {
          return 'default'
        } else {
          return ''
        }
      },
      menuClass () {
        return {
          visible: this.showMenu,
          hidden: !this.showMenu
        }
      },
      menuStyle () {
        return {
          display: this.showMenu ? 'block' : 'none'
        }
      },
      dynamicClass () {
        if (this.loading && this.loadingIconClass) {
          return `icon-right ${this.loadingIconClass}`
        }
        return 'icon'
      }
    },
    watch: {
      searchText (newTerm) {
        this._requestAsyncData(newTerm)
      }
    },
    methods: {
      deleteTextOrItem () {
        if (!this.searchText && this.selectedOption) {
          this.selectItem({})
          this.openOptions()
        }
      },
      openOptions () {
        common.openOptions(this)
      },
      blurInput () {
        common.blurInput(this)
      },
      closeOptions () {
        common.closeOptions(this)
      },
      prevItem () {
        common.prevItem(this)
      },
      nextItem () {
        common.nextItem(this)
      },
      enterItem () {
        common.enterItem(this)
      },
      pointerSet (index) {
        common.pointerSet(this, index)
      },
      pointerAdjust () {
        common.pointerAdjust(this)
      },
      mousedownItem () {
        common.mousedownItem(this)
      },
      selectItem (option) {
        this.searchText = '' // reset text when select item
        this.closeOptions()
        this.$emit('select', option)
      },
      _requestAsyncData (newTerm) {
        if (this.timeoutId) {
          clearTimeout(this.timeoutId)
        }
        this.timeoutId = setTimeout(() => {
          this.loading = true
          this.showMenu = false
          this.httpClient(newTerm).then((arr) => {
            this.options = arr
            this.showMenu = true
          }).catch((err) => {
            this.$emit('ajax-select-error', err)
          }).finally(() => {
            this.timeoutId = null
            this.loading = false
          })
        }, this.delayMillis)
      }
    }
  }
</script>

<style scoped src="semantic-ui-dropdown/dropdown.css"></style>
<style>
  /* custom icon float right*/
  .ui .dropdown.icon-right {
    float: right;
  }
  /* Menu Item Hover */
  .ui.dropdown .menu > .item:hover {
    background: none transparent !important;
  }

  /* Menu Item Hover for Key event */
  .ui.dropdown .menu > .item.current {
    background: rgba(0, 0, 0, 0.05) !important;
  }
</style>
