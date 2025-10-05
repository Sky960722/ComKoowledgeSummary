<template>
  <div>
    <h1>Template Syntax</h1>
    <p>Message: {{ msg }}</p>
    <div>
        <h1 v-bind="objectOfAttrs">{{ msg1 }}</h1>
    </div>
    <div>
        <input type="text" v-model="msg1">
    </div>
  </div>
  <div>
    {{ count }}
    
  </div>
</template>

<script setup>
import { ref } from 'vue'

const msg = ref('Hello Vue!')
const msg1 = ref('文本输入框的动态绑定!')
const objectOfAttrs = {
  id: 'container',
  class: 'wrapper',
  style: 'background-color:green'
}

</script>