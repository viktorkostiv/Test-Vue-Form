<template>
    <button class="btn">
        <slot />
    </button>
</template>

<script>
import { Options, Vue } from 'vue-class-component';

@Options({})
export default class BaseButton extends Vue {}
</script>

<style scoped>
.btn {
  margin-top: 25px;
  outline: none;
  border: none;
  width: 100%;
  background: #0cbe3b;
  text-align: center;
  padding: 10px 10px;
  font-weight: 600;
  color: white;
  border-radius: 4px;
  cursor: pointer;
  transition: .3s ease all;
}

.btn:hover {
  background: #10a237;
}
</style>
