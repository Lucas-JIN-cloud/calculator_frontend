<template>
  <view class="container" :class="currentTheme">
    <!-- Calculator Panel -->
    <view class="calc-panel">
      <view class="display">
        <text class="expression">{{ expression || '0' }}</text>
        <text class="result" v-if="currentResult !== ''">= {{ currentResult }}</text>
        <text class="error-tip" v-if="errorMsg">{{ errorMsg }}</text>
      </view>

      <view class="keys">
        <view class="key func" @tap="appendValue('sin(')">sin</view>
        <view class="key func" @tap="appendValue('cos(')">cos</view>
        <view class="key func" @tap="appendValue('tan(')">tan</view>
        <view class="key func" @tap="appendValue('sqrt(')">√</view>
      
        <view class="key func" @tap="appendValue('log(')">ln</view>
        <view class="key func" @tap="appendValue('pi')">π</view>
        <view class="key func" @tap="appendValue('e')">e</view>
        <view class="key func" @tap="appendValue('%')">%</view>
      
        <view class="key func" @tap="appendValue('(')">(</view>
        <view class="key func" @tap="appendValue(')')">)</view>
        <view class="key func" @tap="appendValue('^')">x^y</view>
        <view class="key func" @tap="clearAll">AC</view>
      
        <view class="key num" @tap="appendValue('7')">7</view>
        <view class="key num" @tap="appendValue('8')">8</view>
        <view class="key num" @tap="appendValue('9')">9</view>
        <view class="key op" @tap="appendValue('/')">÷</view>
      
        <view class="key num" @tap="appendValue('4')">4</view>
        <view class="key num" @tap="appendValue('5')">5</view>
        <view class="key num" @tap="appendValue('6')">6</view>
        <view class="key op" @tap="appendValue('*')">×</view>
      
        <view class="key num" @tap="appendValue('1')">1</view>
        <view class="key num" @tap="appendValue('2')">2</view>
        <view class="key num" @tap="appendValue('3')">3</view>
        <view class="key op" @tap="appendValue('-')">−</view>
      
        <view class="key num zero" @tap="appendValue('0')">0</view>
        <view class="key num" @tap="appendValue('.')">.</view>
        <view class="key op" @tap="appendValue('+')">+</view>
        <view class="key equal" @tap="calculate">=</view>
      </view>
      <!-- Base Conversion Module -->
      <view class="convert-section">
        <text class="section-title">Base Conversion</text>
        <view class="convert-form">
          <input class="convert-input" v-model="convertValue" placeholder="Enter numeric value" />
          <view class="base-select">
            <picker :range="baseOptions" range-key="label" @change="onFromBaseChange">
              <view class="picker-text">Base {{ fromBase }}</view>
            </picker>
            <text class="arrow">→</text>
            <picker :range="baseOptions" range-key="label" @change="onToBaseChange">
              <view class="picker-text">Base {{ toBase }}</view>
            </picker>
          </view>
          <view class="convert-btn" @tap="convertBase">Convert</view>
        </view>
        <text class="convert-result" v-if="convertResult">Result: {{ convertResult }}</text>
        <text class="convert-error" v-if="convertError">{{ convertError }}</text>
      </view>

      <!-- Unit Conversion Module -->
      <view class="convert-section">
        <text class="section-title">Unit Conversion</text>
        <view class="convert-form">
          <picker :range="unitCategories" range-key="label" @change="onCategoryChange">
            <view class="picker-text category-picker">{{ currentCategoryLabel }}</view>
          </picker>
          
          <input class="convert-input" v-model="unitValue" placeholder="Enter value" />
          
          <view class="base-select">
            <picker :range="currentUnitList" range-key="label" @change="onFromUnitChange">
              <view class="picker-text">{{ fromUnitLabel }}</view>
            </picker>
            <text class="arrow">→</text>
            <picker :range="currentUnitList" range-key="label" @change="onToUnitChange">
              <view class="picker-text">{{ toUnitLabel }}</view>
            </picker>
          </view>
          <view class="convert-btn" @tap="convertUnit">Convert</view>
        </view>
        <text class="convert-result" v-if="unitResult">Result: {{ unitResult }}</text>
        <text class="convert-error" v-if="unitError">{{ unitError }}</text>
      </view>
    </view>

    <!-- History Panel -->
    <view class="history-panel">
      <view class="history-header">
        <text class="history-title">Calculation History</text>
        <view class="theme-btn" @tap="switchTheme">🎨 Theme</view>
      </view>

      <!-- Keyboard Shortcuts Panel -->
      <view class="shortcut-section">
        <text class="shortcut-title">Keyboard Shortcuts</text>
        <view class="shortcut-grid">
          <view class="shortcut-item">
            <text class="key-tag">Enter</text>
            <text class="key-desc">Calculate</text>
          </view>
          <view class="shortcut-item">
            <text class="key-tag">Backspace</text>
            <text class="key-desc">Delete char</text>
          </view>
          <view class="shortcut-item">
            <text class="key-tag">Esc</text>
            <text class="key-desc">Clear all</text>
          </view>
          <view class="shortcut-item">
            <text class="key-tag">S / C / T</text>
            <text class="key-desc">sin / cos / tan</text>
          </view>
          <view class="shortcut-item">
            <text class="key-tag">L / R</text>
            <text class="key-desc">ln / √</text>
          </view>
          <view class="shortcut-item">
            <text class="key-tag">P / E</text>
            <text class="key-desc">π / e</text>
          </view>
        </view>
      </view>
      
      <scroll-view class="history-list" scroll-y>
        <view class="history-card" v-for="item in historyList" :key="item.id">
          <view class="card-header">
            <text class="record-id">#{{ item.id }}</text>
            <view class="card-actions">
              <text 
                class="favorite-btn" 
                :class="{ active: item.is_favorite }" 
                @tap="toggleFavorite(item.id)"
              >
                {{ item.is_favorite ? '★' : '☆' }}
              </text>
              <text class="delete-btn" @tap="deleteRecord(item.id)">Delete</text>
            </view>
          </view>
          <view class="record-expr">{{ item.expression }}</view>
          <view class="record-result">= {{ formatNumber(item.result) }}</view>
          <view class="record-time">{{ item.createdAt }}</view>
        </view>
        <view class="empty-tip" v-if="historyList.length === 0">No calculation records</view>
      </scroll-view>
    </view>
  </view>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

const expression = ref('')
const currentResult = ref('')
const errorMsg = ref('')
const historyList = ref([])

// Base Conversion State
const convertValue = ref('')
const fromBase = ref(10)
const toBase = ref(2)
const convertResult = ref('')
const convertError = ref('')
const baseOptions = ref([
  { label: 'Binary (2)', value: 2 },
  { label: 'Octal (8)', value: 8 },
  { label: 'Decimal (10)', value: 10 },
  { label: 'Hex (16)', value: 16 }
])

// Unit Conversion State
const unitValue = ref('')
const unitResult = ref('')
const unitError = ref('')
const unitCategoryIndex = ref(0)
const fromUnitIndex = ref(0)
const toUnitIndex = ref(1)

// Unit Conversion Configuration
const unitConfig = {
  length: {
    label: 'Length',
    units: [
      { label: 'Millimeter (mm)', rate: 0.001 },
      { label: 'Centimeter (cm)', rate: 0.01 },
      { label: 'Meter (m)', rate: 1 },
      { label: 'Kilometer (km)', rate: 1000 },
      { label: 'Inch (in)', rate: 0.0254 },
      { label: 'Foot (ft)', rate: 0.3048 },
      { label: 'Yard (yd)', rate: 0.9144 }
    ]
  },
  weight: {
    label: 'Weight',
    units: [
      { label: 'Gram (g)', rate: 0.001 },
      { label: 'Kilogram (kg)', rate: 1 },
      { label: 'Tonne (t)', rate: 1000 },
      { label: 'Pound (lb)', rate: 0.45359237 },
      { label: 'Ounce (oz)', rate: 0.02834952 }
    ]
  },
  area: {
    label: 'Area',
    units: [
      { label: 'Square centimeter (cm²)', rate: 0.0001 },
      { label: 'Square meter (m²)', rate: 1 },
      { label: 'Square kilometer (km²)', rate: 1000000 },
      { label: 'Hectare (ha)', rate: 10000 },
      { label: 'Square foot (ft²)', rate: 0.09290304 },
      { label: 'Acre', rate: 4046.8564224 }
    ]
  },
  volume: {
    label: 'Volume',
    units: [
      { label: 'Milliliter (mL)', rate: 0.001 },
      { label: 'Liter (L)', rate: 1 },
      { label: 'Cubic meter (m³)', rate: 1000 },
      { label: 'US Gallon (gal)', rate: 3.785411784 }
    ]
  },
  temperature: {
    label: 'Temperature',
    units: [
      { label: 'Celsius (°C)', type: 'c' },
      { label: 'Fahrenheit (°F)', type: 'f' },
      { label: 'Kelvin (K)', type: 'k' }
    ]
  }
}

const unitCategories = computed(() => {
  return Object.values(unitConfig).map(item => ({ label: item.label }))
})

const currentCategoryKey = computed(() => {
  return Object.keys(unitConfig)[unitCategoryIndex.value]
})

const currentCategoryLabel = computed(() => {
  return unitConfig[currentCategoryKey.value].label
})

const currentUnitList = computed(() => {
  return unitConfig[currentCategoryKey.value].units
})

const fromUnitLabel = computed(() => {
  return currentUnitList.value[fromUnitIndex.value].label
})

const toUnitLabel = computed(() => {
  return currentUnitList.value[toUnitIndex.value].label
})

// Theme State
const themeList = ref(['theme-default', 'theme-dark', 'theme-green', 'theme-blue'])
const currentThemeIndex = ref(0)
const currentTheme = computed(() => themeList.value[currentThemeIndex.value])

const API_BASE = 'https://calculator-backend-nshsffpxka.cn-hangzhou.fcapp.run/api'

// Switch Theme
const switchTheme = () => {
  currentThemeIndex.value = (currentThemeIndex.value + 1) % themeList.value.length
}

// Append character to expression
const appendValue = (val) => {
  errorMsg.value = ''
  expression.value += val
}

// Clear all input
const clearAll = () => {
  expression.value = ''
  currentResult.value = ''
  errorMsg.value = ''
}

// Delete last character
const deleteOne = () => {
  expression.value = expression.value.slice(0, -1)
  errorMsg.value = ''
}

// Number formatting
const formatNumber = (val) => {
  const str = String(val)
  return str.endsWith('.0') ? str.slice(0, -2) : str
}

// Send calculation request
const calculate = async () => {
  if (!expression.value.trim()) {
    errorMsg.value = 'Please enter an expression'
    return
  }

  errorMsg.value = ''
  currentResult.value = ''

  try {
    const res = await uni.request({
      url: `${API_BASE}/calculate`,
      method: 'POST',
      header: { 'Content-Type': 'application/json' },
      data: { expression: expression.value }
    })

    if (res.data.success) {
      currentResult.value = formatNumber(res.data.data.result)
      loadHistory()
    } else {
      errorMsg.value = res.data.error || 'Calculation failed'
    }
  } catch (e) {
    errorMsg.value = 'Cannot connect to server'
  }
}

// Load history
const loadHistory = async () => {
  try {
    const res = await uni.request({
      url: `${API_BASE}/history`,
      method: 'GET'
    })
    if (res.data.success) {
      historyList.value = res.data.data
    }
  } catch (e) {}
}

// Toggle favorite
const toggleFavorite = async (id) => {
  try {
    await uni.request({
      url: `${API_BASE}/history/${id}/favorite`,
      method: 'POST'
    })
    loadHistory()
  } catch (e) {}
}

// Delete single record
const deleteRecord = async (id) => {
  try {
    await uni.showModal({
      title: 'Confirm',
      content: 'Delete this record?',
      showCancel: true
    })

    await uni.request({
      url: `${API_BASE}/history/${id}`,
      method: 'DELETE'
    })

    loadHistory()
  } catch (e) {}
}

// Clear all history
const clearAllHistory = async () => {
  if (historyList.value.length === 0) return

  try {
    await uni.showModal({
      title: 'Confirm',
      content: 'Delete all calculation history? This action cannot be undone.',
      showCancel: true
    })

    await uni.request({
      url: `${API_BASE}/history/all`,
      method: 'DELETE'
    })

    loadHistory()
  } catch (e) {}
}

// Base Conversion Handlers
const onFromBaseChange = (e) => {
  fromBase.value = baseOptions.value[e.detail.value].value
  convertResult.value = ''
  convertError.value = ''
}

const onToBaseChange = (e) => {
  toBase.value = baseOptions.value[e.detail.value].value
  convertResult.value = ''
  convertError.value = ''
}

const convertBase = async () => {
  if (!convertValue.value.trim()) {
    convertError.value = 'Please enter a value'
    return
  }
  convertError.value = ''
  convertResult.value = ''

  try {
    const res = await uni.request({
      url: `${API_BASE}/convert/base`,
      method: 'POST',
      header: { 'Content-Type': 'application/json' },
      data: {
        value: convertValue.value,
        fromBase: fromBase.value,
        toBase: toBase.value
      }
    })

    if (res.data.success) {
      convertResult.value = res.data.result
    } else {
      convertError.value = res.data.message
    }
  } catch (e) {
    convertError.value = 'Cannot connect to server'
  }
}

// Unit Conversion Handlers
const onCategoryChange = (e) => {
  unitCategoryIndex.value = e.detail.value
  fromUnitIndex.value = 0
  toUnitIndex.value = 1
  unitResult.value = ''
  unitError.value = ''
}

const onFromUnitChange = (e) => {
  fromUnitIndex.value = e.detail.value
  unitResult.value = ''
}

const onToUnitChange = (e) => {
  toUnitIndex.value = e.detail.value
  unitResult.value = ''
}

const convertUnit = () => {
  if (!unitValue.value.trim()) {
    unitError.value = 'Please enter a value'
    return
  }
  unitError.value = ''
  unitResult.value = ''

  const val = parseFloat(unitValue.value)
  if (isNaN(val)) {
    unitError.value = 'Invalid numeric value'
    return
  }

  const category = currentCategoryKey.value
  const fromUnit = currentUnitList.value[fromUnitIndex.value]
  const toUnit = currentUnitList.value[toUnitIndex.value]

  // Temperature special handling
  if (category === 'temperature') {
    let celsius
    switch (fromUnit.type) {
      case 'c': celsius = val; break
      case 'f': celsius = (val - 32) * 5 / 9; break
      case 'k': celsius = val - 273.15; break
    }
    let target
    switch (toUnit.type) {
      case 'c': target = celsius; break
      case 'f': target = celsius * 9 / 5 + 32; break
      case 'k': target = celsius + 273.15; break
    }
    unitResult.value = target.toFixed(4).replace(/\.?0+$/, '')
    return
  }

  // Standard unit conversion
  const baseValue = val * fromUnit.rate
  const targetValue = baseValue / toUnit.rate
  unitResult.value = targetValue.toFixed(6).replace(/\.?0+$/, '')
}

// Keyboard Shortcuts
const handleKeydown = (e) => {
  const tagName = e.target.tagName.toLowerCase()
  if (tagName === 'input' || tagName === 'textarea') return

  const key = e.key.toLowerCase()

  if (/^[0-9.]$/.test(key)) {
    appendValue(key)
    return
  }

  if (['+', '-', '*', '/', '%', '^', '(', ')'].includes(key)) {
    appendValue(key)
    return
  }

  switch (key) {
    case 'backspace':
      deleteOne()
      break
    case 'enter':
      e.preventDefault()
      calculate()
      break
    case 'escape':
      clearAll()
      break
    case 's':
      appendValue('sin(')
      break
    case 'c':
      appendValue('cos(')
      break
    case 't':
      appendValue('tan(')
      break
    case 'l':
      appendValue('log(')
      break
    case 'p':
      appendValue('pi')
      break
    case 'e':
      appendValue('e')
      break
    case 'r':
      appendValue('sqrt(')
      break
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
  loadHistory()
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<style>
/* Base Layout */
.container {
  display: flex;
  gap: 24px;
  padding: 24px;
  max-width: 960px;
  margin: 0 auto;
  min-height: 100vh;
  box-sizing: border-box;
  transition: background-color 0.3s ease;
}
.calc-panel { flex: 1; }
.history-panel { flex: 1; }

.display {
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 16px;
  min-height: 80px;
  word-break: break-all;
  transition: all 0.3s ease;
}
.expression { font-size: 24px; display: block; }
.result { font-size: 20px; margin-top: 8px; display: block; font-weight: 500; }
.error-tip { color: #ef4444; margin-top: 8px; display: block; font-size: 14px; }

.keys {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 12px;
}
.key {
  height: 56px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  font-size: 18px;
  user-select: none;
  transition: all 0.2s ease;
  cursor: pointer;
}
.key.zero { grid-column: span 2; }

/* Conversion Modules */
.convert-section {
  margin-top: 24px;
  padding-top: 20px;
  border-top: 1px solid;
  transition: all 0.3s ease;
}
.section-title {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 12px;
  display: block;
}
.convert-form {
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.convert-input {
  height: 44px;
  border-radius: 8px;
  padding: 0 12px;
  font-size: 16px;
  box-sizing: border-box;
  transition: all 0.3s ease;
  outline: none;
}
.base-select {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}
.picker-text {
  flex: 1;
  height: 44px;
  line-height: 44px;
  text-align: center;
  border-radius: 8px;
  font-size: 14px;
  transition: all 0.3s ease;
}
.category-picker { width: 100%; }
.arrow { font-size: 18px; }
.convert-btn {
  height: 44px;
  line-height: 44px;
  text-align: center;
  color: #fff;
  border-radius: 8px;
  font-size: 16px;
  transition: background-color 0.3s ease;
  cursor: pointer;
}
.convert-result {
  margin-top: 12px;
  font-size: 18px;
  font-weight: 500;
  display: block;
}
.convert-error {
  margin-top: 8px;
  color: #ef4444;
  font-size: 14px;
  display: block;
}

/* History */
.history-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}
.history-title { font-size: 18px; font-weight: 600; display: block; }

/* Bordered Theme Button */
.theme-btn {
  padding: 6px 14px;
  border-radius: 8px;
  font-size: 13px;
  font-weight: 500;
  user-select: none;
  cursor: pointer;
  border: 2px solid;
  transition: all 0.3s ease;
}

.clear-all-btn { font-size: 13px; color: #ef4444; user-select: none; cursor: pointer; }

/* Shortcut Panel */
.shortcut-section {
  border-radius: 10px;
  padding: 14px;
  margin-bottom: 16px;
  border: 1px solid;
  transition: all 0.3s ease;
}
.shortcut-title {
  font-size: 14px;
  font-weight: 600;
  margin-bottom: 10px;
  display: block;
}
.shortcut-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 8px 12px;
}
.shortcut-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
}
.key-tag {
  display: inline-block;
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 11px;
  font-weight: 600;
  min-width: 50px;
  text-align: center;
  border: 1px solid;
}
.key-desc { opacity: 0.8; }

.history-list { height: 420px; }
.history-card {
  border-radius: 8px;
  padding: 12px;
  margin-bottom: 12px;
  transition: all 0.3s ease;
}
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}
.card-actions { display: flex; gap: 12px; align-items: center; }
.record-id { font-size: 12px; }
.favorite-btn { font-size: 16px; user-select: none; cursor: pointer; }
.favorite-btn.active { color: #f59e0b; }
.delete-btn { color: #ef4444; font-size: 12px; cursor: pointer; }
.record-expr { font-size: 14px; word-break: break-all; }
.record-result { font-size: 16px; font-weight: 500; margin: 4px 0; }
.record-time { font-size: 12px; }
.empty-tip { text-align: center; padding: 40px 0; }

/* ========== Theme 1: Default Light ========== */
.theme-default {
  background-color: #f8fafc;
}
.theme-default .display {
  background-color: #ffffff;
  border: 1px solid #e2e8f0;
}
.theme-default .expression,
.theme-default .result,
.theme-default .history-title,
.theme-default .section-title,
.theme-default .shortcut-title,
.theme-default .record-expr,
.theme-default .record-result,
.theme-default .convert-result {
  color: #1e293b;
}
.theme-default .key {
  background-color: #ffffff;
  border: 1px solid #e2e8f0;
  color: #1e293b;
}
.theme-default .key.func {
  background-color: #f1f5f9;
  color: #475569;
}
.theme-default .key.op {
  background-color: #eff6ff;
  color: #2563eb;
}
.theme-default .key.equal {
  background-color: #2563eb;
  color: #ffffff;
}
.theme-default .history-card,
.theme-default .convert-input,
.theme-default .picker-text,
.theme-default .shortcut-section {
  background-color: #ffffff;
  border-color: #e2e8f0;
  color: #1e293b;
}
.theme-default .convert-btn {
  background-color: #2563eb;
}
.theme-default .theme-btn {
  background-color: #ffffff;
  border-color: #2563eb;
  color: #2563eb;
}
.theme-default .theme-btn:active {
  background-color: #eff6ff;
}
.theme-default .convert-section { border-top-color: #e2e8f0; }
.theme-default .record-id,
.theme-default .record-time,
.theme-default .arrow,
.theme-default .key-desc { color: #64748b; }
.theme-default .favorite-btn { color: #cbd5e1; }
.theme-default .empty-tip { color: #94a3b8; }
.theme-default .key-tag {
  background: #f1f5f9;
  border-color: #cbd5e1;
  color: #475569;
}

/* ========== Theme 2: Dark Mode ========== */
.theme-dark {
  background-color: #0f172a;
}
.theme-dark .display {
  background-color: #1e293b;
  border: 1px solid #334155;
}
.theme-dark .expression,
.theme-dark .result,
.theme-dark .history-title,
.theme-dark .section-title,
.theme-dark .shortcut-title,
.theme-dark .record-expr,
.theme-dark .record-result,
.theme-dark .convert-result {
  color: #f1f5f9;
}
.theme-dark .key {
  background-color: #1e293b;
  border: 1px solid #334155;
  color: #f1f5f9;
}
.theme-dark .key.func {
  background-color: #334155;
  color: #94a3b8;
}
.theme-dark .key.op {
  background-color: #1e3a8a;
  color: #60a5fa;
}
.theme-dark .key.equal {
  background-color: #2563eb;
  color: #ffffff;
}
.theme-dark .history-card,
.theme-dark .convert-input,
.theme-dark .picker-text,
.theme-dark .shortcut-section {
  background-color: #1e293b;
  border-color: #334155;
  color: #f1f5f9;
}
.theme-dark .convert-btn {
  background-color: #2563eb;
}
.theme-dark .theme-btn {
  background-color: #1e293b;
  border-color: #60a5fa;
  color: #60a5fa;
}
.theme-dark .theme-btn:active {
  background-color: #1e3a8a;
}
.theme-dark .convert-section { border-top-color: #334155; }
.theme-dark .record-id,
.theme-dark .record-time,
.theme-dark .arrow,
.theme-dark .key-desc { color: #64748b; }
.theme-dark .favorite-btn { color: #475569; }
.theme-dark .empty-tip { color: #64748b; }
.theme-dark .key-tag {
  background: #334155;
  border-color: #475569;
  color: #cbd5e1;
}

/* ========== Theme 3: Eye Care Green ========== */
.theme-green {
  background-color: #f0fdf4;
}
.theme-green .display {
  background-color: #ffffff;
  border: 1px solid #bbf7d0;
}
.theme-green .expression,
.theme-green .result,
.theme-green .history-title,
.theme-green .section-title,
.theme-green .shortcut-title,
.theme-green .record-expr,
.theme-green .record-result,
.theme-green .convert-result {
  color: #14532d;
}
.theme-green .key {
  background-color: #ffffff;
  border: 1px solid #bbf7d0;
  color: #14532d;
}
.theme-green .key.func {
  background-color: #dcfce7;
  color: #166534;
}
.theme-green .key.op {
  background-color: #dcfce7;
  color: #15803d;
}
.theme-green .key.equal {
  background-color: #16a34a;
  color: #ffffff;
}
.theme-green .history-card,
.theme-green .convert-input,
.theme-green .picker-text,
.theme-green .shortcut-section {
  background-color: #ffffff;
  border-color: #bbf7d0;
  color: #14532d;
}
.theme-green .convert-btn {
  background-color: #16a34a;
}
.theme-green .theme-btn {
  background-color: #ffffff;
  border-color: #16a34a;
  color: #16a34a;
}
.theme-green .theme-btn:active {
  background-color: #dcfce7;
}
.theme-green .convert-section { border-top-color: #bbf7d0; }
.theme-green .record-id,
.theme-green .record-time,
.theme-green .arrow,
.theme-green .key-desc { color: #15803d; }
.theme-green .favorite-btn { color: #86efac; }
.theme-green .empty-tip { color: #4ade80; }
.theme-green .key-tag {
  background: #dcfce7;
  border-color: #86efac;
  color: #166534;
}

/* ========== Theme 4: Deep Blue Tech ========== */
.theme-blue {
  background-color: #eff6ff;
}
.theme-blue .display {
  background-color: #ffffff;
  border: 1px solid #bfdbfe;
}
.theme-blue .expression,
.theme-blue .result,
.theme-blue .history-title,
.theme-blue .section-title,
.theme-blue .shortcut-title,
.theme-blue .record-expr,
.theme-blue .record-result,
.theme-blue .convert-result {
  color: #1e3a8a;
}
.theme-blue .key {
  background-color: #ffffff;
  border: 1px solid #bfdbfe;
  color: #1e3a8a;
}
.theme-blue .key.func {
  background-color: #dbeafe;
  color: #1d4ed8;
}
.theme-blue .key.op {
  background-color: #dbeafe;
  color: #2563eb;
}
.theme-blue .key.equal {
  background-color: #1d4ed8;
  color: #ffffff;
}
.theme-blue .history-card,
.theme-blue .convert-input,
.theme-blue .picker-text,
.theme-blue .shortcut-section {
  background-color: #ffffff;
  border-color: #bfdbfe;
  color: #1e3a8a;
}
.theme-blue .convert-btn {
  background-color: #1d4ed8;
}
.theme-blue .theme-btn {
  background-color: #ffffff;
  border-color: #1d4ed8;
  color: #1d4ed8;
}
.theme-blue .theme-btn:active {
  background-color: #dbeafe;
}
.theme-blue .convert-section { border-top-color: #bfdbfe; }
.theme-blue .record-id,
.theme-blue .record-time,
.theme-blue .arrow,
.theme-blue .key-desc { color: #3b82f6; }
.theme-blue .favorite-btn { color: #93c5fd; }
.theme-blue .empty-tip { color: #60a5fa; }
.theme-blue .key-tag {
  background: #dbeafe;
  border-color: #93c5fd;
  color: #1d4ed8;
}

/* Mobile Responsive */
@media (max-width: 768px) {
  .container { flex-direction: column; }
  .history-list { height: 300px; }
  .shortcut-grid { grid-template-columns: repeat(2, 1fr); }
}
</style>
