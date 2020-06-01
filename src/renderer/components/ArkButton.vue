<template>
  <button
    class="ark-button"
    :class="{
      'ark-button--action': modifiers.includes('action'),
      'ark-button--outline': modifiers.includes('outline')
    }"
  >
    <slot></slot>
  </button>
</template>

<script>
  export default {
    props: {
      modifiers: {
        type: Array,
        default: []
      }
    }
  }
</script>

<style lang="scss" scoped>
  .ark-button {
    position: relative;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    outline: none;
    border: none;
    font-size: 1.6rem;
    font-weight: bold;
    height: 50px;
    width: 150px;
    color: white;
    background-color: rgba(gray, 0.95);
    box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2), 0 5px 10px rgba(0, 0, 0, 0.5);
    transition: 0.2s;
    padding: 0;
    cursor: pointer;

    &::after {
      position: absolute;
      visibility: hidden;
      content: '';
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background-color: transparent;
      transition: background-color 0.2s;
    }

    &:active {
      box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.1),
        0 6px 12px rgba(0, 0, 0, 0.5);
      &::after {
        content: '';
        visibility: visible;
        background-color: rgba(black, 0.1);
      }
    }

    &--outline {
      border: 2px solid #fff;
      font-family: sans-serif;
      box-shadow: 0 10px 15px rgba(0, 0, 0, 0.5);
    }

    &--action {
      background-color: #2bf;
      // box-shadow: 2px 5px 50px -10px rgba(0, 0, 0, 0.5);
    }

    // &--advanced {
    //   background-color: rgba(map-get($colors, 'yellow'), 0.85);
    // }

    &--light {
      background-color: rgba(#eee, 0.85);
    }
  }
</style>
