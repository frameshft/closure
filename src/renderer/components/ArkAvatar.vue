<template>
  <div class="ark-avatar">
    <div class="ark-avatar__splash" :class="`ark-avatar__splash--${level}`">
      <img class="ark-avatar__photo" :src="src" :alt="alt" />
    </div>
    <p class="ark-avatar__name">{{ alt }}</p>
  </div>
</template>

<script>
export default {
  props: {
    modifiers: {
      type: Array,
      default: function() {
        return []
      }
    },
    src: {
      type: String,
      default: ''
    },
    alt: {
      type: String,
      default: ''
    },
    level: {
      type: Number,
      default: 1
    }
  }
}
</script>

<style lang="scss" scoped>
.ark-avatar {
  display: flex;
  flex-direction: column;
  width: 7rem;

  &__splash {
    background-color: transparent;
    transition: 0.2s;
    outline: none;
    display: block;
    padding: 0.5rem 1rem;
    border: 1px solid transparent;
    box-shadow: inset 0 0 20px rgba($color: #9c9c9c, $alpha: 1);
    padding: 0;

    // &:active,
    // &:focus {
    //   border-top: 1px solid #fff;
    //   border-left: 1px solid #fff;
    //   border-right: 1px solid #fff;
    //   box-shadow: inset 0 0 20px rgba($color: #fff, $alpha: 1),
    //     0 5px 10px rgba(0, 0, 0, 0.1);
    //   transform: translateY(-10px);
    // }

    &--1 {
      border-bottom: 4px solid #9c9c9c;
      box-shadow: inset 0 0 20px rgba($color: #9c9c9c, $alpha: 1);
    }

    &--2 {
      border-bottom: 4px solid #d8dd5a;
      box-shadow: inset 0 0 20px rgba($color: #d8dd5a, $alpha: 1);
    }
    &--3 {
      border-bottom: 4px solid #4aabea;
      box-shadow: inset 0 0 20px rgba($color: #4aabea, $alpha: 1);
    }
    &--4 {
      border-bottom: 4px solid #cfc2d1;
      box-shadow: inset 0 0 20px rgba($color: #cfc2d1, $alpha: 1);
    }
    &--5 {
      border-bottom: 4px solid #f1c644;
      box-shadow: inset 0 0 20px rgba($color: #f1c644, $alpha: 1);
    }
    &--6 {
      border-bottom: 4px solid #f6540e;
      box-shadow: inset 0 0 20px rgba($color: #f6540e, $alpha: 1);
    }
  }

  &__photo {
    width: 100%;
  }

  &__name {
    color: #fff;
    font-size: 1.2rem;
    background-color: black;
    text-align: center;
    padding: 0 0.4rem;
  }
}
</style>