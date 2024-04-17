<template>
    <h3>富文本</h3>
    <WangEditor class="WangEditor" @select="getRich" ref="childrenRef" />


</template>
<script setup lang="ts">
import WangEditor from "../components/WangEditor.vue";
import { ref } from 'vue'

//当编辑器的内容更新时，获取该值
const getRich = function (value) {
    // state.introduction = value
    // console.log(value)
}

//获取dom元素
const childrenRef = ref(null);

//调用子组件的方法清空编辑器内容
// childrenRef.value.abc()


</script>

<style lang="scss" scoped></style>