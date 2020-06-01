<template>
  <div class="ark-loader">
    <div class="ark-loader__loading"></div>
    <p class="ark-loader__text">Analyzing</p>
  </div>
</template>

<script>
export default {}
</script>

<style lang="scss" scoped>
.ark-loader {
  position: fixed;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: rgba(0, 0, 0, 0.7);
  z-index: 1000;

  &__loading {
    width: 38px;
    height: 38px;
    background-color: transparent;
    border: 5px solid white;
    transform: rotate(-45deg);
    display: inline-block;
    margin: 66px;

    &::after {
      content: '';
      display: inline-block;
      width: 38px;
      height: 38px;
      background-color: transparent;
      border: 5px solid white;
      position: relative;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      animation-name: ak-radio;
      animation-duration: 0.7s;
      animation-timing-function: ease-out;
      animation-iteration-count: infinite;
    }
  }

  &__text {
    font-size: 2rem;
    color: white;
  }
}
@keyframes ak-radio {
  from {
    width: 38px;
    height: 38px;
    opacity: 1;
  }

  to {
    width: 90px;
    height: 90px;
    opacity: 0;
  }
}
</style>