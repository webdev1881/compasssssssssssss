<template>
  <div class="sales-table-container table-container">
    <div v-if="loading" class="loading-bar">
      <div style="color: white;" class="loading-progress">_</div>
    </div>

    <div v-else class="content">

      <div class="controls-panel" v-if="!loading && !error">
        <div class="sorting-controls">
          <label>Сортировка:</label>
          <select v-model="sortBy" @change="handleSort" :disabled="isAnimating">
            <option value="regionRank">По рангу регионов</option>
            <option value="regionTotalPercent">Регионы по % выполнения</option>
            <option value="storePercent">Магазины по % выполнения</option>
          </select>

        </div>
        <button @click="refreshData" class="refresh-btn" :disabled="loading || isAnimating">
          Обновить
        </button>
      </div>

      <div v-if="!loading && !error">

        <div class="custom-table" :class="{ 'hide-plan-fact': !showPlanFactColumns }">

          <div class="table-header" :style="{ gridTemplateColumns: gridTemplateColumns }">
            <div class="header-cell store-name-column">Регион / Магазин</div>

            <div v-for="(week, weekIndex) in weeks" :key="'week1-' + week.id" class="header-cell week-group"
              :style="`grid-column: ${2 + weekIndex * (showPlanFactColumns ? 11 : 9)} / span ${showPlanFactColumns ? 11 : 9}; grid-row: 1;`">
              {{ week.name }} ({{ week.dateRange }})
            </div>


            <template v-for="(week, weekIndex) in weeks" :key="'week2-' + week.id">
              <div class="header-cell metric-header "
                :style="`grid-column: ${2 + weekIndex * (showPlanFactColumns ? 11 : 9)} / span 2; grid-row: 2;`">
                Загальний бал
              </div>

              <div class="header-cell metric-header h_toggle"
                :style="`grid-column: ${4 + weekIndex * (showPlanFactColumns ? 11 : 9)} / span ${showPlanFactColumns ? 4 : 2}; grid-row: 2;`">
                <div class="vtrg">Виторг<span class="togler">
                    <svg @click="showPlanFactColumns = !showPlanFactColumns" v-if="!showPlanFactColumns"
                      viewBox="0 0 32 32" version="1.1" xmlns="http://www.w3.org/2000/svg"
                      xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:sketch="http://www.bohemiancoding.com/sketch/ns"
                      fill="#555555" stroke="#555555">
                      <g id="SVGRepo_bgCarrier" stroke-width="0"></g>
                      <g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g>
                      <g id="SVGRepo_iconCarrier">
                        <title>plus-square</title>
                        <desc>Created with Sketch Beta.</desc>
                        <defs> </defs>
                        <g id="Page-1" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd"
                          sketch:type="MSPage">
                          <g id="Icon-Set-Filled" sketch:type="MSLayerGroup"
                            transform="translate(-102.000000, -1037.000000)" fill="#1976d2">
                            <path
                              d="M124,1054 L119,1054 L119,1059 C119,1059.55 118.552,1060 118,1060 C117.448,1060 117,1059.55 117,1059 L117,1054 L112,1054 C111.448,1054 111,1053.55 111,1053 C111,1052.45 111.448,1052 112,1052 L117,1052 L117,1047 C117,1046.45 117.448,1046 118,1046 C118.552,1046 119,1046.45 119,1047 L119,1052 L124,1052 C124.552,1052 125,1052.45 125,1053 C125,1053.55 124.552,1054 124,1054 L124,1054 Z M130,1037 L106,1037 C103.791,1037 102,1038.79 102,1041 L102,1065 C102,1067.21 103.791,1069 106,1069 L130,1069 C132.209,1069 134,1067.21 134,1065 L134,1041 C134,1038.79 132.209,1037 130,1037 L130,1037 Z"
                              id="plus-square" sketch:type="MSShapeGroup"> </path>
                          </g>
                        </g>
                      </g>
                    </svg>
                    <svg @click="showPlanFactColumns = !showPlanFactColumns" v-if="showPlanFactColumns"
                      viewBox="0 0 32 32" version="1.1" xmlns="http://www.w3.org/2000/svg"
                      xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:sketch="http://www.bohemiancoding.com/sketch/ns"
                      fill="#000000">
                      <g id="SVGRepo_bgCarrier" stroke-width="0"></g>
                      <g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g>
                      <g id="SVGRepo_iconCarrier">
                        <title>minus-square</title>
                        <desc>Created with Sketch Beta.</desc>
                        <defs> </defs>
                        <g id="Page-1" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd"
                          sketch:type="MSPage">
                          <g id="Icon-Set-Filled" sketch:type="MSLayerGroup"
                            transform="translate(-154.000000, -1037.000000)" fill="#1976d2">
                            <path
                              d="M176,1054 L164,1054 C163.448,1054 163,1053.55 163,1053 C163,1052.45 163.448,1052 164,1052 L176,1052 C176.552,1052 177,1052.45 177,1053 C177,1053.55 176.552,1054 176,1054 L176,1054 Z M182,1037 L158,1037 C155.791,1037 154,1038.79 154,1041 L154,1065 C154,1067.21 155.791,1069 158,1069 L182,1069 C184.209,1069 186,1067.21 186,1065 L186,1041 C186,1038.79 184.209,1037 182,1037 L182,1037 Z"
                              id="minus-square" sketch:type="MSShapeGroup"> </path>
                          </g>
                        </g>
                      </g>
                    </svg>
                  </span></div>
              </div>

              <div class="header-cell metric-header h_loss"
                :style="`grid-column: ${(showPlanFactColumns ? 8 : 6) + weekIndex * (showPlanFactColumns ? 11 : 9)}; grid-row: 2 / 4;`">
                Втрати<br>Списання
              </div>
              <div class="header-cell metric-header h_short"
                :style="`grid-column: ${(showPlanFactColumns ? 9 : 7) + weekIndex * (showPlanFactColumns ? 11 : 9)}; grid-row: 2 / 4;`">
                Недостачі
              </div>
              <div class="header-cell metric-header h_fop"
                :style="`grid-column: ${(showPlanFactColumns ? 10 : 8) + weekIndex * (showPlanFactColumns ? 11 : 9)}; grid-row: 2 / 4;`">
                ФОП
              </div>
              <div class="header-cell metric-header h_minus"
                :style="`grid-column: ${(showPlanFactColumns ? 11 : 9) + weekIndex * (showPlanFactColumns ? 11 : 9)}; grid-row: 2 / 4;`">
                Від'ємні<br>залишки
              </div>
              <div class="header-cell metric-header h_spys"
                :style="`grid-column: ${(showPlanFactColumns ? 12 : 10) + weekIndex * (showPlanFactColumns ? 11 : 9)}; grid-row: 2 / 4;`">
                Не проведені<br>списання
              </div>
            </template>

            <!-- Третий  -->
            <template v-for="(week, weekIndex) in weeks" :key="'week3-' + week.id">
              <div class="header-cell sub-header h_rang"
                :style="`grid-column: ${2 + weekIndex * (showPlanFactColumns ? 11 : 9)}; grid-row: 3;`">
                РАНГ
              </div>
              <div class="header-cell sub-header score-max"
                :style="`grid-column: ${3 + weekIndex * (showPlanFactColumns ? 11 : 9)}; grid-row: 3;`">
                900
              </div>
              <div class="header-cell sub-header score-current"
                :style="`grid-column: ${4 + weekIndex * (showPlanFactColumns ? 11 : 9)}; grid-row: 3;`">
                100
              </div>
              <div class="header-cell sub-header plan-column"
                :style="`grid-column: ${5 + weekIndex * 11}; grid-row: 3;`">
                План
              </div>
              <div class="header-cell sub-header fact-column"
                :style="`grid-column: ${6 + weekIndex * 11}; grid-row: 3;`">
                Факт
              </div>
              <div class="header-cell sub-header h_perc"
                :style="`grid-column: ${(showPlanFactColumns ? 7 : 5) + weekIndex * (showPlanFactColumns ? 11 : 9)}; grid-row: 3;`">
                %
              </div>
            </template>
          </div>


          <div class="table-body regions-body">
            <transition-group name="table-row" tag="div" class="transition-wrapper">
              <div v-for="region in sortedRegions" :key="`region-${region.id}`"
                 :style="{ gridTemplateColumns: gridTemplateColumns }" class="data-row region-row">
                <div class="data-cell region-name">
                  <div class="region-info">
                    <span class="region-indicator" :style="{ backgroundColor: colors[region.name] }"></span>
                    <span class="region-title">{{ region.name }}</span>
                  </div>
                </div>
                <template v-for="week in weeks" :key="week.id">
                  <div class="data-cell region-rank" :style="`grid-column: ${2+(showPlanFactColumns ? 11 : 9)-(showPlanFactColumns ? 11 : 9)}; grid-row: 1;`">{{ getRegionRank(region) }}</div>
                  <!-- <div class="data-cell score-max" :style="`grid-column: ${3 + weekIndex * (showPlanFactColumns ? 11 : 9)}; grid-row: 1;`">{{ 55 }}</div> -->
                  <!-- <div class="data-cell score-current" :style="`grid-column: ${4 + weekIndex * (showPlanFactColumns ? 11 : 9)}; grid-row: 1;`" :class="getScoreClass(getRegionTotalScore(region).current)">{{ 85}}</div> -->
                  <!-- <div class="data-cell plan plan-column" :style="`grid-column: ${5 + weekIndex * (showPlanFactColumns ? 11 : 9)}; grid-row: 1;`">{{ formatNumber(getRegionWeekData(region, week.id).plan) }}</div> -->
                  <!-- <div class="data-cell fact fact-column" :style="`grid-column: ${6 + weekIndex * (showPlanFactColumns ? 11 : 9)}; grid-row: 1;`">{{ formatNumber(getRegionWeekData(region, week.id).fact) }}</div> -->
                  <!-- <div class="data-cell percent" :style="`grid-column: ${7 + weekIndex * (showPlanFactColumns ? 11 : 9)}; grid-row: 1;`" :class="getPercentClass(getRegionWeekData(region, week.id).percent)">{{ getRegionWeekData(region, week.id).percent }}%</div> -->
                  <!-- <div class="data-cell losses" :style="`grid-column: ${1+ (showPlanFactColumns ? 11 : 9)}; grid-row: 1;`">{{ formatNumber(getRegionWeekData(region, week.id).losses) }}</div> -->
                  <!-- <div class="data-cell shortages">{{ formatNumber(getRegionWeekData(region, week.id).shortages) }}</div> -->
                  <!-- <div class="data-cell fop" :style="`grid-column: ${(showPlanFactColumns ? 11 : 9)}; grid-row: 1;`">{{ formatNumber(getRegionWeekData(region, week.id).fop) }}</div> -->
                  <!-- <div class="data-cell shift" :style="`grid-column: ${18-(showPlanFactColumns ? 11 : 9)}; grid-row: 1;`"><span v-if="getRegionWeekData(region, week.id).shiftRemainder" class="status-value negative">{{ getRegionWeekData(region, week.id).shiftRemainder }}</span><span v-else class="status-value">-</span></div> -->
                  <!-- <div class="data-cell unprocessed" :style="`grid-column: ${(showPlanFactColumns ? 11 : 9)}; grid-row: 1;`"><span v-if="getRegionWeekData(region, week.id).unprocessed" class="status-value negative">{{ getRegionWeekData(region, week.id).unprocessed }}</span><span v-else class="status-value">-</span></div> -->
                </template>
              </div>
            </transition-group>

            <!-- Разделитель -->
            <div class="table-separator"></div>


            <div class="table-body stores-body">
              <transition-group name="table-row" tag="div" class="transition-wrapper">
                <div v-for="store in getAllSortedStores()" :key="`store-${store.id}`"
                  :style="{ gridTemplateColumns: gridTemplateColumns }" class="data-row store-row">
                  <!-- {{ store }} -->
                  <div class="data-cell store-name">
                    <div class="store-info">
                      <span class="region-indicator" :style="{ backgroundColor: colors[store.regionName] }"></span>
                      <span class="store-rank-number">{{ store.rank }}</span>
                      <span class="store-title">{{ store.name }}</span>
                    </div>
                  </div>
                  <template v-for="week in weeks" :key="week.id">
                    <div class="data-cell store-rank">{{ store.rank }}</div>
                    <div class="data-cell score-max">{{ 88 }}</div>
                    <div class="data-cell score-current">{{ 99 }}</div>
                    <div class="data-cell plan plan-column">{{ formatNumber(getStoreWeekData(store, week.id).plan) }}</div>
                    <div class="data-cell fact fact-column">{{ formatNumber(getStoreWeekData(store, week.id).fact) }}</div>
                    <div class="data-cell percent" :class="getPercentClass(getStoreWeekData(store, week.id).percent)">
                      {{ getStoreWeekData(store, week.id).percent }}%
                    </div>
                    <div class="data-cell losses">{{ formatNumber(getStoreWeekData(store, week.id).losses) }}</div>
                    <div class="data-cell shortages">{{ formatNumber(getStoreWeekData(store, week.id).shortages) }}
                    </div>
                    <div class="data-cell fop">{{ formatNumber(getStoreWeekData(store, week.id).fop) }}</div>
                    <div class="data-cell shift">
                      <span v-if="getStoreWeekData(store, week.id).shiftRemainder" class="status-value negative">
                        {{ getStoreWeekData(store, week.id).shiftRemainder }}
                      </span>
                      <span v-else class="status-value">-</span>
                    </div>
                    <div class="data-cell unprocessed">
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

        </div>

      </div>

      <div v-else-if="error" class="error">
        <div class="error-icon">⚠️</div>
        <h3>Ошибка загрузки данных</h3>
        <p>{{ error }}</p>
        <button @click="loadData" class="retry-btn">Попробовать снова</button>
      </div>
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
    const showPlanFactColumns = ref(false)

    const showColumn = ref(true)

    const toggleColumn = () => {
      showColumn.value = !showColumn.value
    }

    const colors = ref({
      'Белая Церковь': '#a3352d',
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

        setTimeout(() => {
        }, 400)
        
        salesData.value = data
        loading.value = false

      } catch (err) {
        console.error('Ошибка загрузки данных:', err)
        error.value = err.message || 'Ошибка загрузки данных'
        loading.value = false
      }
    }

    const weeks = computed(() => salesData.value?.weeks || [])
    const regions = computed(() => salesData.value?.regions || [])

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
        return aPercent - bPercent
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
            aValue = getTotalPercentForRegion(b)
            bValue = getTotalPercentForRegion(a)
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
          return sortOrder.value === 'asc' ? bPercent - aPercent : aPercent - bPercent
        })
      } else {
        allStores.sort((a, b) => (a.rank || 0) - (b.rank || 0))
      }

      return allStores
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



    const gridTemplateColumns = computed(() => {
      
      const totalColumns = weeks.value.length * (showPlanFactColumns ? 11 : 9)
      console.log(totalColumns);
      return `240px repeat(${totalColumns}, 1fr)`
    })




    const getColumnPosition = (weekIndex, columnIndex) => {
      let position = 2 // После первой колонки (Регион/Магазин)
      // пред нед
      for (let i = 0; i < weekIndex; i++) {
        position += showPlanFactColumns.value[weeks.value[i].id] ? 11 : 7
      }

      return position + columnIndex
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
  getColumnPosition,
  getRegionWeekData,
  getRegionTotalScore,
  getRegionRank,
  getStoreWeekData,
  getAllSortedStores,
  formatNumber,
  getPercentClass,
  getScoreClass,
  handleSort,
  toggleSortOrder,
  refreshData,
  loadData,
  toggleColumn
}
  }
}
</script>

<style lang="scss" scoped>

// $primary-color: #2196f3;
// $success-color: #2e7d32;
// $warning-color: #f57c00;
// $danger-color: #d32f2f;
// $light-success: #28a745;
// $light-warning: #ffebee;
// $light-danger: #c62828;

// $border-color: #e0e0e0;
// $border-light: #dee2e6;
// $background-light: #f5f5f5;
// $background-white: white;
// $background-hover: #fafafa;
// $background-week: #e3f2fd;
// $background-negative: #ffebee;

// $text-primary: #333;
// $text-secondary: #555;
// $text-week: #0f4478;
// $text-disabled: #6c757d;

// $border-radius: 4px;
// $border-radius-lg: 8px;
// $font-size-sm: 11px;
// $font-size-base: 13px;
// $font-size-lg: 16px;
// $padding-sm: 6px 8px;
// $padding-base: 8px 12px;
// $padding-lg: 15px 20px;

.sales-table-container {
  width: 100%;
  min-height: 100vh;
  max-width: 1900px;
  min-width: 1642px;
  margin: 0 auto;
  padding: 0 20px;
}

.custom-table {
  background: white;
  border-radius: 4px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  overflow-x: auto;
}

.table-header {
  position: sticky;
  top: 0;
  z-index: 10;
  background: white;
  display: grid;
  grid-template-rows: auto auto auto;
}

.store-name-column {
  grid-row: 1 / 4;
  grid-column: 1;
}

.header-row {
  display: contents;
}

.table-header,
.data-row {
  display: grid;
  font-size: 12px;
}

.header-cell {
  padding: 8px;
  text-align: center;
  font-weight: 600;
  color: #333;
  border-right: 1px solid #c7c8c9;
  // box-shadow: 1px 1px 1px 1px red;
  background: #e7e7e7;
  display: flex;
  align-items: center;
  justify-content: center;
}

.week-group {
  grid-column: span v-bind('showPlanFactColumns ? 11 : 9');
  background: #e3f2fd;
  color: #0f4478;
  font-size: 16px;
}

.colspan-2 {
  grid-column: span 2;
}

.colspan-4 {
  grid-column: span 4;
}

.metric-header {
  background: #e7e7e7;
}

.sub-header {
  background: #e7e7e7;
  font-size: 12px;
}

.score-max {
  color: #2e7d32;
}

.empty-cell,
.empty-sub {
  visibility: hidden;
}

.table-body {
  min-height: 100px;
}

.data-row {
  display: grid;
  border-bottom: 1px solid #e0e0e0;
  transition: all 0.3s ease;

  &:hover {
    background: #fafafa;
  }
}

.region-row {
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

.region-name,
.store-name {
  justify-content: flex-start;
  padding-left: 16px;
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

  &.negative {
    background: #ffebee;
    color: #c62828;
  }
}

.table-separator {
  height: 15px;
  background: #e7e7e7;
}

.unprocessed {
  border-right: 1px solid #c7c8c9;
}

.hide-plan-fact {
  .plan-column,
  .fact-column {
    display: none;
    transition: all 0.3s ease;
  }
}

.region-info,
.store-info {
  display: flex;
  align-items: center;
  gap: 6px;
}

.region-indicator {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  display: inline-block;
}

.plan-column,
.fact-column {
  min-width: 76px;
}

.region-rank,.h_rang  {
  min-width: 53px;
}


// .h_rang {
//   min-width: 53px;
// }

.h_perc {
  min-width: 42px;
}

.h_fop {
  min-width: 72px;
}

.score-max,
.store-rank {
  min-width: 46px;
}

.score-current {
  min-width: 36px;
}

.losses, .h_loss {
  min-width: 70px;
}
.fop, .h_fop {
  min-width: 70px;
}
.shortages, .h_short {
  min-width: 70px;
}

.shift, .h_minus {
  min-width: 66px;
}

.unprocessed, .h_spys {
  min-width: 70px;
}

.controls-panel {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: white;
  padding: 15px 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.sorting-controls {
  display: flex;
  align-items: center;
  gap: 10px;

  label {
    font-weight: 600;
    color: #555;
  }

  select {
    padding: 8px 12px;
    border: 1px solid #ddd;
    border-radius: 4px;
    background: white;
  }
}

.sort-order-btn,
.refresh-btn {
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 600;
  transition: background-color 0.3s;
}

.sort-order-btn {
  background: #007bff;
  color: white;

  &:hover {
    background: #0056b3;
  }
}

.refresh-btn {
  background: #28a745;
  color: white;
  display: flex;
  align-items: center;
  gap: 4px;

  &:hover:not(:disabled) {
    background: #218838;
  }

  &:disabled {
    background: #6c757d;
    cursor: not-allowed;
    opacity: 0.6;
  }
}

.loading-bar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: #e0e0e0;
  z-index: 10000;
}

.loading-progress {
  height: 100%;
  background: #2196f3;
  animation: loading 0.5s ease-in-out infinite;
}

@keyframes loading {
  0% {
    width: 0%;
  }
  50% {
    width: 70%;
  }
  100% {
    width: 100%;
  }
}

.content {
  padding: 20px;
  animation: fadeIn 0.25s ease-in;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}



.h_toggle {
  display: flex;
  align-items: center;
  justify-content: left;
}

.vtrg {
  position: relative;
  display: flex;
  align-items: center;
}

.togler {
  position: absolute;
  border-radius: 2px;
  right: -20px;
  top: 2px;
  margin-left: 5px;
  font-size: 10px;
  width: 15px;
  height: 15px;
  color: white;
  background-color: white;
}

.region-row,
.store-row {
  transition: all 0.2s ease;
  transform-origin: center;

  &:hover {
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(1, 80, 133, 0.5);
    z-index: 10;
    position: relative;
  }
}

.error {
  text-align: center;
  padding: 40px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);

  .error-icon {
    font-size: 48px;
    margin-bottom: 16px;
  }

  h3 {
    color: #d32f2f;
    margin-bottom: 8px;
  }

  p {
    color: #555;
    margin-bottom: 20px;
  }

  .retry-btn {
    padding: 8px 12px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-weight: 600;
    transition: background-color 0.3s;
    background: #2196f3;
    color: white;

    &:hover {
      background: #1976d2;
    }
  }
}


@media (prefers-color-scheme: light) {
  .sales-table-container {
    background: #1a1a1a;
    color: #ffffff;
  }
  
  .custom-table {
    background: #2d2d2d;
    
    .header-cell {
      background: #3a3a3a;
      color: #ffffff;
      border-color: #4a4a4a;
    }
    
    .data-row {
      border-color: #4a4a4a;
      
      &:hover {
        background: #3a3a3a;
      }
    }
    
    .data-cell {
      border-color: #4a4a4a;
    }
  }
}

</style>