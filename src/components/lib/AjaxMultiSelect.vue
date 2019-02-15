<template>
  <div class="ui fluid multiple search selection dropdown"
       :class="{ 'active visible':showMenu, 'error': isError, 'disabled': isDisabled }"
       @click="openOptions"
       @focus="openOptions">
    <i class="dropdown" :class="dynamicClass"></i>
    <template v-for="option in selectedOptions">
      <a class="ui label transition visible"
         style="display: inline-block !important;"
         :data-vss-custom-attr="customAttr(option)">
        {{option.text}}<i class="delete icon" @click="deleteItem(option)"></i>
      </a>
    </template>
    <input class="search"
           autocomplete="off"
           tabindex="0"
           v-model="searchText"
           ref="input"
           :style="inputWidth"
           @focus.prevent="openOptions"
           @keyup.esc="closeOptions"
           @blur="blurInput"
           @keydown.up="prevItem"
           @keydown.down="nextItem"
           @keydown.enter.prevent=""
           @keyup.enter.prevent="enterItem"
           @keydown.delete="deleteTextOrLastItem"
    />
    <div class="text"
         :class="textClass" :data-vss-custom-attr="searchTextCustomAttr">{{inputText}}
    </div>
    <div class="menu"
         ref="menu"
         @mousedown.prevent
         :class="menuClass"
         :style="menuStyle"
         @scroll="onScroll"
         tabindex="-1">
      <template v-for="(option, idx) in nonSelectOptions">
        <div class="item"
             :class="{ 'selected': option.selected, 'current': pointer === idx }"
             :data-vss-custom-attr="customAttr(option)"
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
  import differenceBy from 'lodash/differenceBy'
  import last from 'lodash/last'
  import unionWith from 'lodash/unionWith'
  import isEqual from 'lodash/isEqual'
  import reject from 'lodash/reject'
  import common from './common'
  import { baseMixin, commonMixin, optionAwareMixin } from './mixins'

  export default {
    mixins: [baseMixin, commonMixin, optionAwareMixin],
    props: {
      selectedOptions: {
        type: Array
      },
      showMissingOptions: {
        type: Boolean,
        default: false
      },
      cleanSearch: {
        type: Boolean,
        default: true
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
    data () {
      return {
        showMenu: false,
        timeoutId: null,
        searchText: '',
        lastTermSearched: null,
        currentSearch: null,
        httpClientChanged: false,
        originalValues: [],
        loading: false,
        exhaustedResults: false,
        mousedownState: false, // mousedown on option menu
        pointer: 0,
        allOptions: []
      }
    },
    created () {
      this.originalValues = this.selectedOptions
      this._requestAsyncData({ term: '', delayMillis: 0, toggleShow: false })
    },
    computed: {
      optionsWithOriginal () {
        if (this.originalValues.length && this.showMissingOptions) {
          const missingValues = differenceBy(this.originalValues, this.allOptions)
          if (missingValues.length) {
            return this.allOptions.concat(missingValues)
          }
        }
        return this.allOptions
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
          return this.placeholder
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
      dynamicClass () {
        if (this.loading && this.loadingIconClass) {
          return `icon-right ${this.loadingIconClass}`
        }
        return 'icon'
      },
      textClass () {
        if (this.placeholder) {
          return 'default'
        } else {
          return ''
        }
      },
      inputWidth () {
        return {
          width: ((this.searchText.length + 1) * 8) + 20 + 'px'
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
      nonSelectOptions () {
        return differenceBy(this.optionsWithOriginal, this.selectedOptions, 'value')
      }
    },
    watch: {
      value () {
        this._requestAsyncData({ term: '', delayMillis: 0, toggleShow: false })
      },
      searchText (newTerm) {
        this.exhaustedResults = false
        if (this.$refs.input === document.activeElement) {
          this._requestAsyncData({ term: newTerm })
        }
      },
      disabled (newDisabled) {
        if (!newDisabled) {
          this._requestAsyncData({ term: '', delayMillis: 0, toggleShow: false })
        }
      },
      httpClient(newValue) {
        this.httpClientChanged = true
      }
    },
    methods: {
      deleteTextOrLastItem () {
        if (!this.searchText && this.selectedOptions.length > 0) {
          this.deleteItem(last(this.selectedOptions))
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
        this.closeOptions()
        this.openOptions()
      },
      nextItem () {
        common.nextItem(this)
        this.closeOptions()
        this.openOptions()
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
        const selectedOptions = unionWith(this.selectedOptions, [option], isEqual)
        // this.closeOptions()
        this.mousedownState = false
        this.searchText = ''
        this.$emit('select', selectedOptions, option, 'insert')
      },
      onScroll (scrollEvent) {
        const element = scrollEvent.target
        const offset = element.scrollTop + element.offsetHeight
        const height = element.scrollHeight

        if (offset >= height && !this.exhaustedResults && !this.loading) {
          this._requestAsyncData({ term: this.searchText, delayMillis: 0, page: this.page + 1, toggleShow: false })
        }
      },
      deleteItem (option) {
        const selectedOptions = reject(this.selectedOptions, option)
        this.$emit('select', selectedOptions, option, 'delete')
      },
      accentsTidy (s) {
        var r = s.toString().toLowerCase()
        r = r.replace(new RegExp('[àáâãäå]', 'g'), 'a')
        r = r.replace(new RegExp('æ', 'g'), 'ae')
        r = r.replace(new RegExp('ç', 'g'), 'c')
        r = r.replace(new RegExp('[èéêë]', 'g'), 'e')
        r = r.replace(new RegExp('[ìíîï]', 'g'), 'i')
        r = r.replace(new RegExp('ñ', 'g'), 'n')
        r = r.replace(new RegExp('[òóôõö]', 'g'), 'o')
        r = r.replace(new RegExp('œ', 'g'), 'oe')
        r = r.replace(new RegExp('[ùúûü]', 'g'), 'u')
        r = r.replace(new RegExp('[ýÿ]', 'g'), 'y')
        return r
      },
      _requestAsyncData ({ term, delayMillis = this.delayMillis, toggleShow = true, page = 0 }) {
        if ((term !== this.lastTermSearched && term !== this.currentSearch) || page > 0 || this.httpClientChanged) {
          this.currentSearch = term
          this.httpClientChanged = false
          if (this.timeoutId) {
            clearTimeout(this.timeoutId)
          }
          this.timeoutId = setTimeout(() => {
            this.loading = true
            if (toggleShow) {
              this.showMenu = false
            }
            this.httpClient(term, page).then((arr) => {
              this.page = page
              if (page === 0) {
                this.allOptions = arr
              } else if (arr.length) {
                this.allOptions = this.allOptions.concat(arr)
              } else {
                this.exhaustedResults = true
              }
              if (toggleShow) {
                this.showMenu = true
              }
            }).catch((err) => {
              this.$emit('ajax-select-error', err)
            }).finally(() => {
              this.timeoutId = null
              this.loading = false
              this.lastTermSearched = term
              this.currentSearch = null
            })
          }, delayMillis)
        }
      }
    }
  }
</script>
<style scoped src="semantic-ui-label/label.css"></style>
<style scoped src="semantic-ui-dropdown/dropdown.css"></style>
<style>
  /* Menu Item Hover for Key event */
  .ui.dropdown .menu > .item.current {
    background: rgba(0, 0, 0, 0.05);
  }
</style>
