<template>
    <div v-if="visible" class="modal-overlay" @click="close">
        <div 
            class="modal"
            :style="{
                width: width + 'px',
                height: height + 'px',
                left: x + 'px',
                top: y + 'px'
            }"
            @click.stop
        >
            <!-- 拖拽區域 -->
            <div class="modal-header" @mousedown="startDrag">
                <span>可拖拽縮放模態框</span>
                <button @click="close" class="close-btn">×</button>
            </div>

            <!-- 內容區域 -->
            <div class="modal-content">
                <slot></slot>
            </div>

            <!-- 縮放控制點 -->
            <div 
                class="resizer" 
                v-for="dir in ['top','bottom','left','right','top-left','top-right','bottom-left','bottom-right']" 
                :key="dir"
                :class="`resizer-${dir}`"
                @mousedown="startResize($event, dir)"
            >
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const props = defineProps({
    visible: Boolean
})

const emit = defineEmits(['update:visible'])

// 模態框狀態
const width = ref(400)
const height = ref(300)
const x = ref(0)
const y = ref(0)

// 拖拽狀態
const isDragging = ref(false)
const dragStartX = ref(0)
const dragStartY = ref(0)
const modalStartX = ref(0)
const modalStartY = ref(0)

// 縮放狀態
const isResizing = ref(false)
const resizeDirection = ref('')
const resizeStartX = ref(0)
const resizeStartY = ref(0)
const resizeStartWidth = ref(0)
const resizeStartHeight = ref(0)
const resizeStartLeft = ref(0)
const resizeStartTop = ref(0)

// 居中顯示
const centerModal = () => {
    const windowWidth = window.innerWidth
    const windowHeight = window.innerHeight
    x.value = (windowWidth - width.value) / 2
    y.value = (windowHeight - height.value) / 2
}

// 關閉模態框
const close = () => {
    emit('update:visible', false)
}

// 開始拖拽
const startDrag = (e: MouseEvent) => {
    isDragging.value = true
    dragStartX.value = e.clientX
    dragStartY.value = e.clientY
    modalStartX.value = x.value
    modalStartY.value = y.value
    
    e.preventDefault()
}

// 處理拖拽
const handleDrag = (e: MouseEvent) => {
    if (!isDragging.value) return
    
    const deltaX = e.clientX - dragStartX.value
    const deltaY = e.clientY - dragStartY.value
    
    // 計算新位置
    let newX = modalStartX.value + deltaX
    let newY = modalStartY.value + deltaY
    
    // 邊界檢查
    const windowWidth = window.innerWidth
    const windowHeight = window.innerHeight
    
    if (newX < 0) newX = 0
    if (newY < 0) newY = 0
    if (newX + width.value > windowWidth) newX = windowWidth - width.value
    if (newY + height.value > windowHeight) newY = windowHeight - height.value
    
    x.value = newX
    y.value = newY
}

// 停止拖拽
const stopDrag = () => {
    isDragging.value = false
}

// 開始縮放
const startResize = (e: MouseEvent, direction: string) => {
    isResizing.value = true
    resizeDirection.value = direction
    resizeStartX.value = e.clientX
    resizeStartY.value = e.clientY
    resizeStartWidth.value = width.value
    resizeStartHeight.value = height.value
    resizeStartLeft.value = x.value
    resizeStartTop.value = y.value
    
    e.preventDefault()
}

// 處理縮放
const handleResize = (e: MouseEvent) => {
    if (!isResizing.value) return
    
    const deltaX = e.clientX - resizeStartX.value
    const deltaY = e.clientY - resizeStartY.value
    
    // 最小尺寸
    const minWidth = 200
    const minHeight = 150
    
    // 根據方向調整大小和位置
    switch (resizeDirection.value) {
        case 'right': // 右側
            width.value = Math.max(minWidth, resizeStartWidth.value + deltaX)
            break
        case 'left': // 左側
            const newWidth = Math.max(minWidth, resizeStartWidth.value - deltaX)
            if (newWidth > minWidth) {
                x.value = resizeStartLeft.value + deltaX
                width.value = newWidth
            }
            break
        case 'bottom': // 底部
            height.value = Math.max(minHeight, resizeStartHeight.value + deltaY)
            break
        case 'top': // 頂部
            const newHeight = Math.max(minHeight, resizeStartHeight.value - deltaY)
            if (newHeight > minHeight) {
                y.value = resizeStartTop.value + deltaY
                height.value = newHeight
            }
            break
        case 'top-right': // 右上
            const newWidthTR = Math.max(minWidth, resizeStartWidth.value + deltaX)
            const newHeightTR = Math.max(minHeight, resizeStartHeight.value - deltaY)
            
            if (newHeightTR > minHeight) {
                y.value = resizeStartTop.value + deltaY
                height.value = newHeightTR
            }
            width.value = newWidthTR
            break
        case 'top-left': // 左上
            const newWidthTL = Math.max(minWidth, resizeStartWidth.value - deltaX)
            const newHeightTL = Math.max(minHeight, resizeStartHeight.value - deltaY)
            
            if (newWidthTL > minWidth) {
                x.value = resizeStartLeft.value + deltaX
                width.value = newWidthTL
            }
            if (newHeightTL > minHeight) {
                y.value = resizeStartTop.value + deltaY
                height.value = newHeightTL
            }
            break
        case 'bottom-right': // 右下
            width.value = Math.max(minWidth, resizeStartWidth.value + deltaX)
            height.value = Math.max(minHeight, resizeStartHeight.value + deltaY)
            break
        case 'bottom-left': // 左下
            const newWidthBL = Math.max(minWidth, resizeStartWidth.value - deltaX)
            const newHeightBL = Math.max(minHeight, resizeStartHeight.value + deltaY)
            
            if (newWidthBL > minWidth) {
                x.value = resizeStartLeft.value + deltaX
                width.value = newWidthBL
            }
            height.value = newHeightBL
            break
    }
}

// 停止縮放
const stopResize = () => {
    isResizing.value = false
    resizeDirection.value = ''
}

// 監聽全局事件
onMounted(() => {
    window.addEventListener('mousemove', handleDrag)
    window.addEventListener('mouseup', stopDrag)
    window.addEventListener('mousemove', handleResize)
    window.addEventListener('mouseup', stopResize)
    
    // 初始化居中
    centerModal()
})

onUnmounted(() => {
    window.removeEventListener('mousemove', handleDrag)
    window.removeEventListener('mouseup', stopDrag)
    window.removeEventListener('mousemove', handleResize)
    window.removeEventListener('mouseup', stopResize)
})
</script>

<style scoped>
.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
}

.modal {
    position: absolute;
    background: white;
    border-radius: 8px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
    overflow: hidden;
    min-width: 200px;
    min-height: 150px;
}

.modal-header {
    padding: 12px 16px;
    background: #3498db;
    color: white;
    cursor: move;
    display: flex;
    justify-content: space-between;
    align-items: center;
    user-select: none;
}

.close-btn {
    background: none;
    border: none;
    color: white;
    font-size: 20px;
    cursor: pointer;
    padding: 0;
    width: 24px;
    height: 24px;
    border-radius: 50%;
}

.close-btn:hover {
    background: rgba(255, 255, 255, 0.2);
}

.modal-content {
    padding: 16px;
    height: calc(100% - 48px);
    overflow: auto;
}

.resizer {
    position: absolute;
    background: transparent;
    z-index: 10;
}

.resizer-top {
    top: 0;
    left: 0;
    width: 100%;
    height: 4px;
    cursor: n-resize;
}

.resizer-bottom {
    bottom: 0;
    left: 0;
    width: 100%;
    height: 4px;
    cursor: s-resize;
}

.resizer-left {
    top: 0;
    left: 0;
    width: 4px;
    height: 100%;
    cursor: w-resize;
}

.resizer-right {
    top: 0;
    right: 0;
    width: 4px;
    height: 100%;
    cursor: e-resize;
}

.resizer-top-left {
    top: 0;
    left: 0;
    width: 8px;
    height: 8px;
    cursor: nw-resize;
}

.resizer-top-right {
    top: 0;
    right: 0;
    width: 8px;
    height: 8px;
    cursor: ne-resize;
}

.resizer-bottom-left {
    bottom: 0;
    left: 0;
    width: 8px;
    height: 8px;
    cursor: sw-resize;
}

.resizer-bottom-right {
    bottom: 0;
    right: 0;
    width: 8px;
    height: 8px;
    cursor: se-resize;
}
</style>
