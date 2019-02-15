<template>

  <div>
    <h3>Multi</h3>
    <multi-example />

    <h3>Multi with custom attribute to allow custom css</h3>
    <custom-multi-example />

    <h3>With ajax</h3>
    <with-ajax />

    <h3>With missing value</h3>
    <with-missing-example />
  </div>

</template>

<script>
  import MultiExample from './_Multi'
  import WithAjax from './_Ajax'
  import CustomMultiExample from './_CustomMulti'
  import WithMissingExample from './_WithMissing'

  export default {
    components: {
      CustomMultiExample,
      MultiExample,
      WithAjax,
      WithMissingExample
    }
  }
</script>
