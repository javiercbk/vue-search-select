<template>
  <div class="flexbox">
    <div class="flex-content">
      <div>
        <button type="button" @click="reset" class="btn btn-info btn-sm">reset</button>
      </div>
      <div>
        <button type="button" @click="selectOption" class="btn btn-info btn-sm">option select from parent</button>
      </div>
      <div>
        <ajax-multi-select
          :show-missing-options="true"
          :selected-options="items"
          placeholder="select item"
          :http-client="httpClient"
          loading-icon-class="fas fa-spinner fa-spin"
          :delay-millis="800"
          @select="onSelect">
        </ajax-multi-select>
      </div>
    </div>
    <div class="flex-result">
      <table class="ui celled table">
        <thead>
        <tr>
          <th>value</th>
          <th>text</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="item in items" :key="item.value">
          <td>{{item.value}}</td>
          <td>{{item.text}}</td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
  import { AjaxMultiSelect } from '../../lib'

  const options = [
    { value: '1', text: 'aa' + ' - ' + '1' },
    { value: '2', text: 'ab' + ' - ' + '2' },
    { value: '3', text: 'bc' + ' - ' + '3' },
    { value: '4', text: 'cd' + ' - ' + '4' },
    { value: '5', text: 'de' + ' - ' + '5' },
    { value: '6', text: 'ef' + ' - ' + '6' },
    { value: '10', text: 'ef' + ' - ' + '10' },
    { value: '11', text: 'ef' + ' - ' + '11' },
    { value: '12', text: 'ef' + ' - ' + '12' },
    { value: '13', text: 'down case' + ' - ' + 'testcase' },
    { value: '14', text: 'camel case' + ' - ' + 'testCase' },
    { value: '15', text: 'Capitalize case' + ' - ' + 'Testcase' },
    { value: '16', text: 'more a' + ' - ' + '1' },
    { value: '17', text: 'more a' + ' - ' + '2' },
    { value: '18', text: 'more a' + ' - ' + '3' },
    { value: '19', text: 'more a' + ' - ' + '4' },
    { value: '20', text: 'more a' + ' - ' + '5' },
    { value: '21', text: 'more a' + ' - ' + '6' },
    { value: '22', text: 'more a' + ' - ' + '7' },
    { value: '23', text: 'more a' + ' - ' + '8' },
    { value: '24', text: 'more a' + ' - ' + '9' },
    { value: '25', text: 'more a' + ' - ' + '10' },
    { value: '26', text: 'more a' + ' - ' + '10' },
    { value: '27', text: 'more a' + ' - ' + '10' },
    { value: '28', text: 'more a' + ' - ' + '10' },
    { value: '29', text: 'more a' + ' - ' + '10' },
    { value: '24', text: 'more a' + ' - ' + '10' },
    { value: '24', text: 'more a' + ' - ' + '10' },
    { value: '24', text: 'more a' + ' - ' + '10' },
    { value: '24', text: 'more a' + ' - ' + '10' },
    { value: '24', text: 'more a' + ' - ' + '10' },
    { value: '24', text: 'more a' + ' - ' + '10' },
    { value: '24', text: 'more a' + ' - ' + '10' },
    { value: '24', text: 'more a' + ' - ' + '10' },
    { value: '24', text: 'more a' + ' - ' + '10' },
    { value: '24', text: 'more a' + ' - ' + '10' },
    { value: '24', text: 'more a' + ' - ' + '9' }
  ]

  const perPage = 10

  export default {
    data () {
      return {
        searchText: '', // If value is falsy, reset searchText & searchItem
        items: []
      }
    },
    methods: {
      onSelect (items) {
        this.items = items
      },
      reset () {
        this.items = []
      },
      selectOption () {
        // select option from parent component
        this.items = options
      },
      httpClient (newTerm, page) {
        const from = page * perPage
        const to = from + perPage
        console.log(`Searching for ${newTerm}`)
        return new Promise((resolve) => {
          setTimeout(() => {
            const allResults = options.filter(o => o.text.indexOf(newTerm) !== -1)
            if (allResults.length > perPage) {
              resolve(allResults.slice(from, to))
            } else {
              resolve(allResults)
            }
          }, 1000)
        })
      }
    },
    components: {
      AjaxMultiSelect
    }
  }
</script>
