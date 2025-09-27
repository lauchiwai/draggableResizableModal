<template>
    <div class="button-container">
        <button class="bnt" @click="modelOpen =  true">
            model open
        </button>
    </div>

    <DraggableResizableModal
        v-model:open="modelOpen"
    />

    <!-- <MyDraggableResizableModal 
        v-model:visible="modelOpen"
    /> -->
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DraggableResizableModal from '@/components/DraggableResizableModal.vue'
import MyDraggableResizableModal from '@/components/MyDraggableResizableModal.vue'
const modelOpen = ref(false);
</script>

<style scoped lang="scss">
.button-container{
    height: 100vh;
    width: 100vw;
    display: flex;
    align-items: center;
    justify-content: center;
    
    .bnt {
        border: none;
        background-color: #3698b4;
        height: 30%;
        width: 30%;
        min-height: 100px;
        min-width: 100px;
        border-radius: 5%;
        color: white;
        &:hover {
            cursor: pointer;
            background-color: #0a3862;
        }
    }
}


</style>
