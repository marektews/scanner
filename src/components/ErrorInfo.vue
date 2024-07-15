<template>
    <div class="alert alert-danger">
        <h6>{{ props.label }}:</h6>
        <div class="text">{{ props.text }}</div>
    </div>
</template>

<script setup>
const props = defineProps(['text', 'label'])
</script>

<style scoped>
.text {
    font-size: 1.4rem;
}
</style>