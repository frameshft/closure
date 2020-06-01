<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
  export default {
    name: 'closure'
  }
</script>

<style lang="scss">
  html {
    font-size: 62.5%;
    -webkit-overflow-scrolling: touch;
    box-sizing: border-box;
  }

  body {
    font-family: 'SF Pro Display', -apple-system, BlinkMacSystemFont, sans-serif;
    font-weight: 400;
    background-color: #0d0e12;
    min-height: 100%;
    margin: 0;
    padding: 0;
    border: 0;
    font-size: 100%;
    vertical-align: baseline;
  }

  html,
  body,
  #app {
    height: 100%;
    margin: 0;
    padding: 0;
    border: 0;
    vertical-align: baseline;
  }

  #app > div {
    min-height: 100%;
    display: flex;
    flex-direction: column;
  }

  *,
  *:before,
  *:after {
    margin: 0;
    padding: 0;
    box-sizing: inherit;
  }

  body,
  h1,
  h2,
  h3,
  h4,
  h5,
  h6,
  p,
  ol,
  ul {
    font-weight: normal;
  }

  ol,
  ul {
    list-style: none;
  }

  img {
    max-width: 100%;
    height: auto;
  }
</style>
