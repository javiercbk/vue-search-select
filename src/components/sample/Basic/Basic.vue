<template>
  <div>
    <h3>Basic</h3>
    <basic-example />

    <h3>isError prop</h3>
    <error-prop-example />

    <h3>isDisabled prop</h3>
    <disabled-prop-example />

    <h3>With Ajax</h3>
    <ajax-example />

    <h3>Custom attribute to allow custom css</h3>
    <custom-attr-example />

    <h3>With missing value</h3>
    <with-missing-example />


  </div>
</template>

<script>
  import AjaxExample from './_Ajax'
  import BasicExample from './_Basic'
  import CustomAttrExample from './_CustomAttr'
  import ErrorPropExample from './_ErrorProp'
  import DisabledPropExample from './_DisabledProp'
  import WithMissingExample from './_WithMissing'

  export default {
    components: {
      AjaxExample,
      BasicExample,
      CustomAttrExample,
      ErrorPropExample,
      DisabledPropExample,
      WithMissingExample
    }
  }
</script>
