<script>
export default {
  props: ['active'],
  data(){
    return {}
  },
  mounted(){
    // this.$emit('init', { module: this.module, target: this.name })
  }
}
</script>

<style lang="sass" scoped>
  // FX
  .fx
    > h4 
      margin: 0
  // Params
  .params
    display: grid
    grid-template-columns: 1fr 1fr
  // CONTROL
  .control
    display: block
    padding: 10px 20px
    border: dotted 1px #ddd
    h4
      font-weight: bold
      font-size: 12px
      text-align: left
      margin: 0
      margin-bottom: 6px
      span
        display: inline-block
        &:first-child
          text-transform: uppercase
        &:last-child
          padding-left: 10px
    input
      &[type="range"]
        display: block
        width: 100%
        height: 6px
    &--select
      display: grid
      grid-template-columns: 2fr 3fr
      h4,
      select
        line-height: 30px
      h4
        margin-bottom: 0
      select
        padding: 0 8px

</style>
