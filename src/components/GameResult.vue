<template>
  <div
    class="gameover text-9xl fixed top-11 w-screen h-screen flex justify-center items-center z-50"
  >
    <div
      class="gameover__content flex justify-center items-center rounded-lg uppercase tracking-wider text-center"
    >
      <slot>
        <!-- Slot -->
      </slot>
    </div>
  </div>
</template>

<script setup lang="ts"></script>

<style lang="scss" scoped>
.gameover {
  &__content {
    width: clamp(30rem, 90%, 55rem);
    min-height: 55rem;
    background-color: transparent;
    backdrop-filter: blur(0.5rem);
    animation: tracking-in-expand-fwd 0.8s cubic-bezier(0.215, 0.61, 0.355, 1)
      both;

    @keyframes tracking-in-expand-fwd {
      0% {
        letter-spacing: -5rem;
        -webkit-transform: translateZ(-70rem);
        transform: translateZ(-70rem);
        opacity: 0;
      }
      40% {
        opacity: 0.6;
      }
      100% {
        -webkit-transform: translateZ(0);
        transform: translateZ(0);
        opacity: 1;
      }
    }
  }
}
</style>
