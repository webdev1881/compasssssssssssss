<template>
    <div class="table-container">
        <!-- Тогглы -->
        <div class="controls">
            <label v-for="col in columns" :key="col.key"><input type="checkbox" v-model="visible[col.key]" />{{
                col.label }}</label>
        </div>

        <!-- Таблица -->
        <div class="table">

            <!-- Шапка: уровень 1 -->
            <div class="row header top">
                <div class="cell static" rowspan="2">#</div>
                <div class="cell group" :style="{ width: dynamicRowWidth }">Основная информация</div>
            </div>

            <!-- Шапка: уровень 2 -->
            <div class="row header bottom">
                <div class="cell static" rowspan="2">#</div>
                <div v-for="col in columns" :key="col.key" class="cell dynamic header" :style="getStyle(col.key)">{{
                    col.label }}</div>
            </div>

            <!-- Данные -->
            <div class="row" v-for="(row, rowIndex) in rows" :key="rowIndex">
                <div class="cell static">{{ rowIndex + 1 }}</div>
                <div v-for="col in columns" :key="`${rowIndex}-${col.key}`" class="cell dynamic"
                    :style="getStyle(col.key)"> {{ row[col.key] }} </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { reactive, computed } from 'vue'

const columns = [{ key: 'name', label: 'Name' },{ key: 'age', label: 'Age' },{ key: 'city', label: 'City' },{ key: 'position', label: 'Position' },]
const rows = [{ name: 'Alice', age: 28, city: 'Kyiv', position: 'Engineer' },{ name: 'Bob', age: 32, city: 'Lviv', position: 'Designer' },{ name: 'Charlie', age: 24, city: 'Odesa', position: 'Manager' },]
const visible = reactive( columns.reduce((acc, col) => { acc[col.key] = true
    return acc
}, {}))
const visibleKeys = computed(() => columns.filter(col => visible[col.key]).map(col => col.key))
const dynamicRowWidth = computed(() => { const total = visibleKeys.value.length
    return total > 0 ? '100%' : '0%'
})
function getStyle(key) {
    const total = visibleKeys.value.length
    const isVisible = visible[key]
    const width = isVisible ? `${100 / total}%` : '0%'
    return {width, opacity: isVisible ? 1 : 0, transition: 'width 0.3s ease, opacity 0.3s ease', overflow: 'hidden',}
}
</script>

<style scoped>
.table-container {
    padding: 1rem;
    font-family: sans-serif;
}

.controls {
    margin-bottom: 1rem;
    display: flex;
    gap: 1rem;
    flex-wrap: wrap;
}

.table {
    width: 100%;
    border: 1px solid #ccc;
    box-sizing: border-box;
}

.row {
    display: flex;
    width: 100%;
}

.cell {
    padding: 8px;
    border: 1px solid #ccc;
    box-sizing: border-box;
    text-align: center;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

.cell.static {
    width: 60px;
    flex-shrink: 0;
    background: #f9f9f9;
    font-weight: bold;
}

.cell.dynamic {
    text-align: center;
}

.header {
    background: #eee;
    font-weight: bold;
}

.top .group {
    display: block;
    text-align: center;
    background: #ddd;
}

.bottom .cell.dynamic {
    background: #f0f0f0;
}
</style>
