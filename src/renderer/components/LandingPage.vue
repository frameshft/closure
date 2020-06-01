<template>
  <div>
    <ark-card>
      <template v-slot:title>
        Controls
      </template>
      <ark-button :modifiers="['outline']">Select Arknights</ark-button>
      <ark-button :modifiers="['action']">Analyze</ark-button>
    </ark-card>
  </div>
</template>

<script>
  import ArkCard from './ArkCard'
  import ArkButton from './ArkButton'
  export default {
    components: {
      ArkCard,
      ArkButton
    }
  }
</script>

<style lang="scss" scoped>
  body {
    $background: #333;
    background-color: $background;
  }
</style>
