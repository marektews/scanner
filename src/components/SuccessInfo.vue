<template>
    <div class="alert alert-success">
        <div><FontAwesomeIcon :icon="faThumbsUp" /> OK</div>
    </div>
</template>

<script setup>
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { faThumbsUp } from '@fortawesome/free-solid-svg-icons';
</script>

<style scoped>
.alert {
    font-size: 1.4rem;
    text-align: center;
}
</style>