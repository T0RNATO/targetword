<script setup>
defineProps(["letter", "i"])
</script>

<template>
<div class='square' :data-i='i'>
    <span>{{letter}}</span>
</div>
</template>

<style scoped>
.square {
    background: var(--white);
    width: calc(33% - 6px);
    display: flex;
    justify-content: center;
    align-items: center;
    aspect-ratio: 1/1;
    cursor: pointer;
    border-radius: 5px;
}
.square[data-i='4'] {
    background: #e8e895;
}
.square.selected {
    background: #9fe39f;
}
span {
    color: var(--bg);
    font-size: 40px;
}
</style>