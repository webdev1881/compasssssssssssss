<template>
  <div class="sales-table-container table-container">
    <div v-if="loading" class="loading-bar">
      <div class="loading-progress"></div>
    </div>

    <div class="controls-panel" v-if="!loading && !error">
      <div class="sorting-controls">
        <label>Сортировка:</label>
        <select v-model="sortBy" @change="handleSort" :disabled="isAnimating">
          <option value="regionRank">По рангу регионов</option>
          <option value="regionTotalPercent">Регионы по % выполнения</option>
          <option value="storePercent">Магазины по % выполнения</option>
        </select>

        <button @click="toggleSortOrder" class="sort-order-btn" :class="{ 'animating': isAnimating }">
          {{ sortOrder === 'asc' ? '↑' : '↓' }}
        </button>

        <button @click="refreshData" class="refresh-btn" :disabled="loading || isAnimating">
          🔄 Обновить
        </button>
      </div>

      <div class="week-toggles">
        <div v-for="week in weeks" :key="'toggle-' + week.id" class="week-toggle-group">
          <span class="week-label">{{ week.name }}:</span>
          <button 
            @click="toggleWeekColumns(week.id)" 
            class="toggle-columns-btn"
            :class="{ 'is-hidden': !showPlanFactColumns[week.id] }"
          >
            {{ showPlanFactColumns[week.id] ? '👁️ Скрыть План/Факт' : '👁️‍🗨️ Показать План/Факт' }}
          </button>
        </div>
      </div>
    </div>

    <div v-if="!loading && !error" class="custom-table">
      <!-- Заголовки таблицы -->
      <div class="table-header" :style="{ gridTemplateColumns: gridTemplateColumns }">
        <!-- Все ячейки заголовков в одной сетке -->
        <div class="header-cell store-name-column">Регион / Магазин</div>
        
        <!-- Первый уровень - недели -->
        <div v-for="(week, weekIndex) in weeks" :key="'week1-' + week.id" 
             class="header-cell week-group"
             :style="`grid-column: ${2 + weekIndex * 11} / span 11; grid-row: 1;`">
          {{ week.name }} ({{ week.dateRange }})
        </div>
        
        <!-- Второй уровень -->
        <template v-for="(week, weekIndex) in weeks" :key="'week2-' + week.id">
          <div class="header-cell metric-header" 
               :style="`grid-column: ${2 + weekIndex * 11} / span 2; grid-row: 2;`">
            Загальний бал
          </div>
          <div class="header-cell metric-header" 
               :style="`grid-column: ${4 + weekIndex * 11} / span 4; grid-row: 2;`">
            Виторг
          </div>
          <div class="header-cell metric-header" 
               :style="`grid-column: ${8 + weekIndex * 11}; grid-row: 2 / 4;`">
            Втрати<br>Списання
          </div>
          <div class="header-cell metric-header" 
               :style="`grid-column: ${9 + weekIndex * 11}; grid-row: 2 / 4;`">
            Недостачі
          </div>
          <div class="header-cell metric-header" 
               :style="`grid-column: ${10 + weekIndex * 11}; grid-row: 2 / 4;`">
            ФОП
          </div>
          <div class="header-cell metric-header" 
               :style="`grid-column: ${11 + weekIndex * 11}; grid-row: 2 / 4;`">
            Від'ємні<br>залишки
          </div>
          <div class="header-cell metric-header" 
               :style="`grid-column: ${12 + weekIndex * 11}; grid-row: 2 / 4;`">
            Не проведені<br>списання
          </div>
        </template>
        
        <!-- Третий уровень -->
        <template v-for="(week, weekIndex) in weeks" :key="'week3-' + week.id">
          <div class="header-cell sub-header" 
               :style="`grid-column: ${2 + weekIndex * 11}; grid-row: 3;`">
            РАНГ
          </div>
          <div class="header-cell sub-header score-max" 
               :style="`grid-column: ${3 + weekIndex * 11}; grid-row: 3;`">
            900
          </div>
          <div class="header-cell sub-header score-current" 
               :style="`grid-column: ${4 + weekIndex * 11}; grid-row: 3;`">
            100
          </div>
          <div :class="`header-cell sub-header plan-column grid-col-${5 + weekIndex * 11}`" 
               :style="`grid-column: ${5 + weekIndex * 11}; grid-row: 3;`">
            План
          </div>
          <div :class="`header-cell sub-header fact-column grid-col-${6 + weekIndex * 11}`" 
               :style="`grid-column: ${6 + weekIndex * 11}; grid-row: 3;`">
            Факт
          </div>
          <div class="header-cell sub-header" 
               :style="`grid-column: ${7 + weekIndex * 11}; grid-row: 3;`">
            %
          </div>
        </template>
      </div>

      <!-- Таблица регионов -->
      <div class="table-body regions-body">
        <transition-group name="table-row" tag="div" class="transition-wrapper">
          <div v-for="region in sortedRegions" 
               :key="`region-${region.id}`" 
               class="data-row region-row"
               :style="{ gridTemplateColumns: gridTemplateColumns }">
            
            <div class="data-cell region-name">
              <div class="region-info">
                <span class="region-indicator" :style="{ backgroundColor: colors[region.name] }"></span>
                <span class="region-title">{{ region.name }}</span>
              </div>
            </div>
            
            <template v-for="(week, weekIndex) in weeks" :key="week.id">
              <div class="data-cell region-rank" :style="`grid-column: ${getGridColumn(weekIndex, 'rank')}`">
                {{ getRegionRank(region) }}
              </div>
              <div class="data-cell score-max" :style="`grid-column: ${getGridColumn(weekIndex, 'scoreMax')}`">
                {{ '88' }}
              </div>
              <!-- :class="getScoreClass(getRegionTotalScore(region).current)" -->
              <div class="data-cell score-current" 
                   :style="`grid-column: ${getGridColumn(weekIndex, 'scoreCurrent')}`">
                {{ '99' }}
              </div>
              <div :class="`data-cell plan grid-col-${getGridColumn(weekIndex, 'plan')}`" 
                   :style="`grid-column: ${getGridColumn(weekIndex, 'plan')}`">
                {{ formatNumber(getRegionWeekData(region, week.id).plan) }}
              </div>
              <div :class="`data-cell fact grid-col-${getGridColumn(weekIndex, 'fact')}`" 
                   :style="`grid-column: ${getGridColumn(weekIndex, 'fact')}`">
                {{ formatNumber(getRegionWeekData(region, week.id).fact) }}
              </div>
              <div class="data-cell percent" 
                   :class="getPercentClass(getRegionWeekData(region, week.id).percent)"
                   :style="`grid-column: ${getGridColumn(weekIndex, 'percent')}`">
                {{ getRegionWeekData(region, week.id).percent }}%
              </div>
              <div class="data-cell losses" :style="`grid-column: ${getGridColumn(weekIndex, 'losses')}`">
                {{ formatNumber(getRegionWeekData(region, week.id).losses) }}
              </div>
              <div class="data-cell shortages" :style="`grid-column: ${getGridColumn(weekIndex, 'shortages')}`">
                {{ formatNumber(getRegionWeekData(region, week.id).shortages) }}
              </div>
              <div class="data-cell fop" :style="`grid-column: ${getGridColumn(weekIndex, 'fop')}`">
                {{ formatNumber(getRegionWeekData(region, week.id).fop) }}
              </div>
              <div class="data-cell shift" :style="`grid-column: ${getGridColumn(weekIndex, 'shift')}`">
                <span v-if="getRegionWeekData(region, week.id).shiftRemainder" class="status-value negative">
                  {{ getRegionWeekData(region, week.id).shiftRemainder }}
                </span>
                <span v-else class="status-value">-</span>
              </div>
              <div class="data-cell unprocessed" :style="`grid-column: ${getGridColumn(weekIndex, 'unprocessed')}`">
                <span v-if="getRegionWeekData(region, week.id).unprocessed" class="status-value negative">
                  {{ getRegionWeekData(region, week.id).unprocessed }}
                </span>
                <span v-else class="status-value">-</span>
              </div>
            </template>
          </div>
        </transition-group>
      </div>

      <!-- Разделитель между таблицами -->
      <div class="table-separator"></div>

      <!-- Таблица магазинов -->
      <div class="table-body stores-body">
        <transition-group name="table-row" tag="div" class="transition-wrapper">
          <div v-for="store in getAllSortedStores()" 
               :key="`store-${store.id}`" 
               class="data-row store-row"
               :style="{ gridTemplateColumns: gridTemplateColumns }">
            
            <div class="data-cell store-name">
              <div class="store-info">
                <span class="region-indicator" :style="{ backgroundColor: colors[store.regionName] }"></span>
                <span class="store-rank-number">{{ store.rank }}</span>
                <span class="store-title">{{ store.name }}</span>
              </div>
            </div>
            
            <template v-for="(week, weekIndex) in weeks" :key="week.id">
              <div class="data-cell store-rank" :style="`grid-column: ${getGridColumn(weekIndex, 'rank')}`">
                {{ store.rank }}
              </div>
              <div class="data-cell score-max" :style="`grid-column: ${getGridColumn(weekIndex, 'scoreMax')}`">
                {{ '5' }}
              </div>
              <!-- :class="getScoreClass(store.totalScore.current)" -->
              <div class="data-cell score-current" 
                   :style="`grid-column: ${getGridColumn(weekIndex, 'scoreCurrent')}`">
                {{ '7' }}
              </div>
              <div :class="`data-cell plan grid-col-${getGridColumn(weekIndex, 'plan')}`" 
                   :style="`grid-column: ${getGridColumn(weekIndex, 'plan')}`">
                {{ formatNumber(getStoreWeekData(store, week.id).plan) }}
              </div>
              <div :class="`data-cell fact grid-col-${getGridColumn(weekIndex, 'fact')}`" 
                   :style="`grid-column: ${getGridColumn(weekIndex, 'fact')}`">
                {{ formatNumber(getStoreWeekData(store, week.id).fact) }}
              </div>
              <div class="data-cell percent" 
                   :class="getPercentClass(getStoreWeekData(store, week.id).percent)"
                   :style="`grid-column: ${getGridColumn(weekIndex, 'percent')}`">
                {{ getStoreWeekData(store, week.id).percent }}%
              </div>
              <div class="data-cell losses" :style="`grid-column: ${getGridColumn(weekIndex, 'losses')}`">
                {{ formatNumber(getStoreWeekData(store, week.id).losses) }}
              </div>
              <div class="data-cell shortages" :style="`grid-column: ${getGridColumn(weekIndex, 'shortages')}`">
                {{ formatNumber(getStoreWeekData(store, week.id).shortages) }}
              </div>
              <div class="data-cell fop" :style="`grid-column: ${getGridColumn(weekIndex, 'fop')}`">
                {{ formatNumber(getStoreWeekData(store, week.id).fop) }}
              </div>
              <div class="data-cell shift" :style="`grid-column: ${getGridColumn(weekIndex, 'shift')}`">
                <span v-if="getStoreWeekData(store, week.id).shiftRemainder" class="status-value negative">
                  {{ getStoreWeekData(store, week.id).shiftRemainder }}
                </span>
                <span v-else class="status-value">-</span>
              </div>
              <div class="data-cell unprocessed" :style="`grid-column: ${getGridColumn(weekIndex, 'unprocessed')}`">
                <span v-if="getStoreWeekData(store, week.id).unprocessed" class="status-value negative">
                  {{ getStoreWeekData(store, week.id).unprocessed }}
                </span>
                <span v-else class="status-value">-</span>
              </div>
            </template>
          </div>
        </transition-group>
      </div>
    </div>

    <div v-else-if="loading" class="loading">
      <div class="loading-spinner"></div>
      <p>Загрузка данных...</p>
    </div>

    <div v-else-if="error" class="error">
      <div class="error-icon">⚠️</div>
      <h3>Ошибка загрузки данных</h3>
      <p>{{ error }}</p>
      <button @click="loadData" class="retry-btn">Попробовать снова</button>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  name: 'SalesTable',
  setup() {
    const loading = ref(true)
    const error = ref(null)
    const salesData = ref(null)
    const sortBy = ref('regionRank')
    const sortOrder = ref('asc')
    const isAnimating = ref(false)
    const showPlanFactColumns = ref({})
    
    const colors = ref({
      'Белая Церковь': '#f44336',
      'Днепр': '#ffc107',
      'Киев': '#4caf50',
      'Харьков': '#2196f3',
    })

    const loadData = async () => {
      try {
        loading.value = true
        error.value = null
        const response = await fetch('/sales-data.json')

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`)
        }

        const data = await response.json()
        
        if (!data.weeks || !data.regions) {
          throw new Error('Неверная структура данных')
        }

        salesData.value = data
        
        // Инициализируем showPlanFactColumns для каждой недели
        data.weeks.forEach(week => {
          showPlanFactColumns.value[week.id] = true
        })
        
      } catch (err) {
        console.error('Ошибка загрузки данных:', err)
        error.value = err.message || 'Ошибка загрузки данных'
      } finally {
        loading.value = false
      }
    }

    const weeks = computed(() => salesData.value?.weeks || [])
    const regions = computed(() => salesData.value?.regions || [])

    const gridTemplateColumns = computed(() => {
      const totalColumns = weeks.value.length * 11
      return `230px repeat(${totalColumns}, 1fr)`
    })

    const getGridColumn = (weekIndex, columnType) => {
      const baseColumn = 2 + weekIndex * 11
      const columnOffsets = {
        'rank': 0,
        'scoreMax': 1,
        'scoreCurrent': 2,
        'plan': 3,
        'fact': 4,
        'percent': 5,
        'losses': 6,
        'shortages': 7,
        'fop': 8,
        'shift': 9,
        'unprocessed': 10
      }
      return baseColumn + columnOffsets[columnType]
    }

    const getRegionWeekData = (region, weekId) => {
      const stores = region.stores || []
      const weekData = {
        plan: 0,
        fact: 0,
        losses: 0,
        shortages: 0,
        fop: 0,
        shiftRemainder: 0,
        unprocessed: 0
      }

      stores.forEach(store => {
        const storeWeekData = store.weeklyData?.find(w => w.weekId === weekId)
        if (storeWeekData) {
          weekData.plan += storeWeekData.plan || 0
          weekData.fact += storeWeekData.fact || 0
          weekData.losses += storeWeekData.losses || 0
          weekData.shortages += storeWeekData.shortages || 0
          weekData.fop += storeWeekData.fop || 0
          weekData.shiftRemainder += storeWeekData.shiftRemainder || 0
          weekData.unprocessed += storeWeekData.unprocessed || 0
        }
      })

      weekData.percent = weekData.plan > 0 ? Math.round((weekData.fact / weekData.plan) * 100) : 0

      return weekData
    }

    const getRegionTotalScore = (region) => {
      const stores = region.stores || []
      let maxScore = 0
      let currentScore = 0

      stores.forEach(store => {
        if (store.totalScore) {
          maxScore += store.totalScore.max || 0
          currentScore += store.totalScore.current || 0
        }
      })

      return { max: maxScore, current: currentScore }
    }

    const getRegionRank = (region) => {
      const allRegions = [...regions.value]
      allRegions.sort((a, b) => {
        const aPercent = getTotalPercentForRegion(a)
        const bPercent = getTotalPercentForRegion(b)
        return bPercent - aPercent
      })
      
      return allRegions.findIndex(r => r.id === region.id) + 1
    }

    const getTotalPlanForRegion = (region) => {
      let totalPlan = 0
      const stores = region.stores || []
      
      stores.forEach(store => {
        if (store.weeklyData) {
          store.weeklyData.forEach(week => {
            totalPlan += week.plan || 0
          })
        }
      })
      
      return totalPlan
    }

    const getTotalFactForRegion = (region) => {
      let totalFact = 0
      const stores = region.stores || []
      
      stores.forEach(store => {
        if (store.weeklyData) {
          store.weeklyData.forEach(week => {
            totalFact += week.fact || 0
          })
        }
      })
      
      return totalFact
    }

    const getTotalPercentForRegion = (region) => {
      const totalPlan = getTotalPlanForRegion(region)
      const totalFact = getTotalFactForRegion(region)
      return totalPlan > 0 ? Math.round((totalFact / totalPlan) * 100) : 0
    }

    const sortedRegions = computed(() => {
      if (!regions.value) return []

      const sorted = [...regions.value]

      sorted.sort((a, b) => {
        let aValue, bValue

        switch (sortBy.value) {
          case 'regionRank':
            aValue = getRegionRank(a)
            bValue = getRegionRank(b)
            break
          case 'regionTotalPercent':
            aValue = getTotalPercentForRegion(a)
            bValue = getTotalPercentForRegion(b)
            break
          default:
            return 0
        }

        return sortOrder.value === 'asc' ? aValue - bValue : bValue - aValue
      })

      return sorted
    })

    const getStoreWeekData = (store, weekId) => {
      if (!store || !store.weeklyData) {
        return { plan: 0, fact: 0, percent: 0, losses: 0, shortages: 0, fop: 0, shiftRemainder: 0, unprocessed: 0 }
      }

      const weekData = store.weeklyData.find(w => w.weekId === weekId)
      return weekData || { plan: 0, fact: 0, percent: 0, losses: 0, shortages: 0, fop: 0, shiftRemainder: 0, unprocessed: 0 }
    }

    const getTotalPercentForStore = (store) => {
      if (!store || !store.weeklyData) return 0
      
      let totalPlan = 0
      let totalFact = 0
      
      store.weeklyData.forEach(week => {
        totalPlan += week.plan || 0
        totalFact += week.fact || 0
      })
      
      return totalPlan > 0 ? Math.round((totalFact / totalPlan) * 100) : 0
    }

    const getAllSortedStores = () => {
      const allStores = []

      regions.value.forEach(region => {
        const stores = region.stores || []
        stores.forEach(store => {
          allStores.push({
            ...store,
            regionName: region.name
          })
        })
      })

      if (sortBy.value === 'storePercent') {
        allStores.sort((a, b) => {
          const aPercent = getTotalPercentForStore(a)
          const bPercent = getTotalPercentForStore(b)
          return sortOrder.value === 'asc' ? aPercent - bPercent : bPercent - aPercent
        })
      } else {
        allStores.sort((a, b) => (a.rank || 0) - (b.rank || 0))
      }

      return allStores
    }

    const toggleWeekColumns = (weekId) => {
      // Проверяем, что объект инициализирован
      if (!showPlanFactColumns.value || typeof showPlanFactColumns.value !== 'object') {
        showPlanFactColumns.value = {}
        weeks.value.forEach(week => {
          showPlanFactColumns.value[week.id] = true
        })
      }
      
      const weekIndex = weeks.value.findIndex(w => w.id === weekId)
      const planColumnIndex = 5 + weekIndex * 11
      const factColumnIndex = 6 + weekIndex * 11
      
      const planColumns = document.querySelectorAll(`.grid-col-${planColumnIndex}`)
      const factColumns = document.querySelectorAll(`.grid-col-${factColumnIndex}`)
      
      // Проверяем текущее состояние
      const isCurrentlyShown = showPlanFactColumns.value[weekId] !== false
      
      if (isCurrentlyShown) {
        // Скрываем с анимацией
        planColumns.forEach(col => {
          col.style.transition = 'width 0.3s ease-out, opacity 0.3s ease-out'
          col.style.width = '0'
          col.style.opacity = '0'
          col.style.overflow = 'hidden'
          col.style.padding = '0'
          col.style.borderWidth = '0'
        })
        
        factColumns.forEach(col => {
          col.style.transition = 'width 0.3s ease-out, opacity 0.3s ease-out'
          col.style.width = '0'
          col.style.opacity = '0'
          col.style.overflow = 'hidden'
          col.style.padding = '0'
          col.style.borderWidth = '0'
        })
        
        setTimeout(() => {
          showPlanFactColumns.value[weekId] = false
        }, 300)
      } else {
        // Показываем с анимацией
        showPlanFactColumns.value[weekId] = true
        
        planColumns.forEach(col => {
          col.style.transition = 'width 0.3s ease-out, opacity 0.3s ease-out'
          col.style.width = ''
          col.style.opacity = ''
          col.style.overflow = ''
          col.style.padding = ''
          col.style.borderWidth = ''
        })
        
        factColumns.forEach(col => {
          col.style.transition = 'width 0.3s ease-out, opacity 0.3s ease-out'
          col.style.width = ''
          col.style.opacity = ''
          col.style.overflow = ''
          col.style.padding = ''
          col.style.borderWidth = ''
        })
      }
    }

    const formatNumber = (number) => {
      if (number === null || number === undefined || isNaN(number)) {
        return '0'
      }
      return new Intl.NumberFormat('ru-RU').format(number)
    }

    const getPercentClass = (percent) => {
      if (percent === null || percent === undefined || isNaN(percent)) {
        return 'danger'
      }
      if (percent >= 70) return 'success'
      if (percent >= 50) return 'warning'
      return 'danger'
    }

    const getScoreClass = (score) => {
      if (score === null || score === undefined || isNaN(score)) {
        return 'danger'
      }
      if (score >= 90) return 'success'
      if (score >= 70) return 'warning'
      return 'danger'
    }

    const handleSort = () => {
      isAnimating.value = true
      setTimeout(() => {
        isAnimating.value = false
      }, 600)
    }

    const toggleSortOrder = () => {
      isAnimating.value = true
      sortOrder.value = sortOrder.value === 'asc' ? 'desc' : 'asc'
      setTimeout(() => {
        isAnimating.value = false
      }, 600)
    }

    const refreshData = async () => {
      await loadData()
    }

    onMounted(() => {
      loadData()
    })

    return {
      loading,
      error,
      weeks,
      regions,
      sortedRegions,
      sortBy,
      sortOrder,
      isAnimating,
      colors,
      showPlanFactColumns,
      gridTemplateColumns,
      getGridColumn,
      getRegionWeekData,
      getRegionTotalScore,
      getRegionRank,
      getStoreWeekData,
      getAllSortedStores,
      toggleWeekColumns,
      formatNumber,
      getPercentClass,
      getScoreClass,
      handleSort,
      toggleSortOrder,
      refreshData,
      loadData
    }
  }
}
</script>

<style scoped>
.sales-table-container {
  width: 100%;
  min-height: 100vh;
  background: #f5f5f5;
}

.loading-bar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: #e0e0e0;
  z-index: 1000;
}

.loading-progress {
  height: 100%;
  background: #2196f3;
  animation: loading 1.5s ease-in-out infinite;
}

@keyframes loading {
  0% { width: 0%; }
  50% { width: 70%; }
  100% { width: 100%; }
}

.controls-panel {
  padding: 20px;
  background: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.sorting-controls {
  display: flex;
  align-items: center;
  gap: 10px;
}

.week-toggles {
  display: flex;
  gap: 20px;
  margin-left: 20px;
}

.week-toggle-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.week-label {
  font-weight: 500;
  color: #606266;
}

.toggle-columns-btn {
  padding: 6px 12px;
  border: 1px solid #dcdfe6;
  background: white;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s;
}

.toggle-columns-btn:hover {
  color: #409eff;
  border-color: #409eff;
}

.toggle-columns-btn.is-hidden {
  opacity: 0.7;
}

.sort-order-btn {
  padding: 6px 12px;
  border: 1px solid #dcdfe6;
  background: white;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: all 0.3s;
}

.sort-order-btn:hover {
  color: #409eff;
  border-color: #409eff;
}

.refresh-btn {
  padding: 6px 12px;
  border: 1px solid #dcdfe6;
  background: white;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s;
}

.refresh-btn:hover {
  color: #409eff;
  border-color: #409eff;
}

.refresh-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* Основная структура таблицы */
.custom-table {
  background: white;
  margin: 20px;
  border-radius: 4px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  overflow-x: auto;
}

/* Заголовки таблицы */
.table-header {
  position: sticky;
  top: 0;
  z-index: 10;
  background: white;
  display: grid;
  grid-template-rows: auto auto auto;
}

.header-cell {
  padding: 8px;
  text-align: center;
  font-weight: 600;
  color: #333;
  border: 1px solid #dee2e6;
  background: #f8f9fa;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 13px;
}

.store-name-column {
  grid-row: 1 / 4;
  grid-column: 1;
}

.week-group {
  background: #e3f2fd;
  color: #1976d2;
  font-size: 15px;
  font-weight: 600;
}

.metric-header {
  background: #f5f5f5;
  font-size: 12px;
}

.sub-header {
  background: #fafafa;
  font-size: 12px;
  font-weight: 500;
}

.score-max {
  background: #e8f5e9;
  color: #2e7d32;
}

/* Тело таблицы */
.table-body {
  min-height: 100px;
}

.data-row {
  display: grid;
  border-bottom: 1px solid #e0e0e0;
  transition: all 0.3s ease;
}

.data-row:hover {
  background: #fafafa;
}

.region-row {
  background: #f8f9fa;
  font-weight: 600;
}

.data-cell {
  padding: 6px 8px;
  border-right: 1px solid #e0e0e0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 13px;
}

/* Стили для различных типов ячеек */
.region-name,
.store-name {
  justify-content: flex-start;
  padding-left: 16px;
}

.region-info,
.store-info {
  display: flex;
  align-items: center;
  gap: 8px;
}

.region-indicator {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  flex-shrink: 0;
}

.region-title {
  font-weight: 600;
  color: #303133;
}

.store-rank-number {
  font-weight: 600;
  color: #666;
  min-width: 20px;
}

.store-title {
  color: #333;
}

.plan,
.fact,
.losses,
.shortages,
.fop {
  text-align: right;
  justify-content: flex-end;
  font-family: monospace;
}

.percent {
  font-weight: 600;
}

.score-current {
  font-weight: 600;
}

/* Цветовые классы */
.success {
  color: #2e7d32;
}

.warning {
  color: #f57c00;
}

.danger {
  color: #d32f2f;
}

.status-value {
  display: inline-block;
  padding: 2px 6px;
  border-radius: 2px;
  font-size: 11px;
  font-weight: 600;
}

.status-value.negative {
  background: #ffebee;
  color: #c62828;
}

/* Разделитель между таблицами */
.table-separator {
  height: 20px;
  background: #f5f5f5;
}

/* Состояния загрузки и ошибки */
.loading,
.error {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 400px;
  padding: 40px;
}

.loading-spinner {
  width: 40px;
  height: 40px;
  border: 3px solid #f3f3f3;
  border-top: 3px solid #409eff;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.error-icon {
  font-size: 48px;
  margin-bottom: 16px;
}

.retry-btn {
  margin-top: 16px;
  padding: 8px 16px;
  background: #409eff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
}

.retry-btn:hover {
  background: #66b1ff;
}

/* Анимации для transition-group */
.table-row-enter-active,
.table-row-leave-active {
  transition: all 0.5s ease;
}

.table-row-enter-from,
.table-row-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

.table-row-move {
  transition: transform 0.5s ease;
}
</style>