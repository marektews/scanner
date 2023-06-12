<template>
    <div class="vrn-border">
        <div class="vrn-title">
            {{ props.label }}
        </div>

        <div class="vrn-text">
            {{ props.text }}
        </div>
    </div>
</template>

<script setup>
const props = defineProps(['text', 'label'])
</script>

<style scoped>
.vrn-border {
    border-radius: 9pt;
    border-color: black;
    border-width: 3pt;
    border-style: solid;
    background-color: white;
}

.vrn-title {
    background-color: black;
    color: white;
    font-size: 10pt;
    padding: 3pt 9pt;
}

.vrn-text {
    color: black;
    text-align: center;
    text-transform: uppercase;
    font-size: 32pt;
    font-weight: bold;
}
</style>