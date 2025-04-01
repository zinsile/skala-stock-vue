<template>
  <section class="stock-list">
    <h2 class="title">📈 현재 주식 목록</h2>

    <ul class="stock-cards">
      <li
        v-for="stock in stocks"
        :key="stock.stockName"
        class="stock-item"
      >
        <div class="stock-info">
          <span class="stock-name">{{ stock.stockName }}</span>
          <span class="stock-price">{{ stock.stockPrice.toLocaleString() }} 원</span>
        </div>
        <div class="stock-actions">
          <button
            class="btn-buy"
            @click="buy(stock.stockName)"
          >
            매수
          </button>
          <button
            class="btn-sell"
            @click="sell(stock.stockName)"
          >
            매도
          </button>
        </div>
      </li>
    </ul>

    <!-- ✅ 새로운 주식 종목 추가 폼 -->
    <div class="add-stock-form">
      <h3 class="form-title">➕ 새로운 주식 종목 추가</h3>
      <div class="form-fields">
        <input
          v-model="newStockName"
          placeholder="종목명"
        />
        <input
          v-model.number="newStockPrice"
          type="number"
          placeholder="주 당 가격"
        />
        <button @click="addStock">추가</button>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const newStockName = ref('')
const newStockPrice = ref(0)
const props = defineProps(['stocks'])
const emit = defineEmits(['buy', 'sell', 'refresh'])

const buy = async (stockName) => {
  const quantity = prompt('몇 주를 매수할까요?')
  if (!quantity || isNaN(parseInt(quantity)) || parseInt(quantity) <= 0) {
    return alert('유효한 수량을 입력하세요.')
  }
  
  // App.vue의 handleBuy로 이벤트 발생
  emit('buy', null, stockName, parseInt(quantity))
}

const sell = async (stockName) => {
  const quantity = prompt('몇 주를 매도할까요?')
  if (!quantity || isNaN(parseInt(quantity)) || parseInt(quantity) <= 0) {
    return alert('유효한 수량을 입력하세요.')
  }

  // App.vue의 handleSell로 이벤트 발생
  emit('sell', stockName, parseInt(quantity))
}

const addStock = async () => {
  if (!newStockName.value || newStockPrice.value <= 0) {
    return alert('종목명과 유효한 가격을 입력하세요.')
  }

  try {
    await axios.post('http://localhost:8080/api/stocks', null, {
      params: {
        name: newStockName.value,
        price: newStockPrice.value
      }
    })

    alert('주식이 추가되었습니다!')
    newStockName.value = ''
    newStockPrice.value = 0

    emit('refresh')
  } catch (error) {
    if (error.response && error.response.status === 409) {
      alert('이미 존재하는 종목입니다.')
    } else {
      alert('주식 추가 중 오류가 발생했습니다.')
    }
  }
}
</script>

<style scoped>
.stock-list {
  font-family: 'Pretendard', 'Noto Sans KR', sans-serif;
}

.title {
  font-size: 20px;
  font-weight: 600;
  margin-bottom: 16px;
  color: #1f2937;
}

.stock-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 24px;
}

.stock-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border: 1px solid #e5e7eb;
  border-radius: 10px;
  padding: 12px 16px;
  background: #ffffff;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.03);
}

.stock-info {
  display: flex;
  flex-direction: column;
}

.stock-name {
  font-weight: 600;
  color: #111827;
  font-size: 16px;
}

.stock-price {
  color: #2563eb;
  font-size: 14px;
}

.stock-actions button {
  margin-left: 8px;
}

.btn-buy {
  padding: 6px 12px;
  background-color: #4ade80;
  color: #065f46;
  border: 1px solid #86efac;
  border-radius: 6px;
  font-size: 14px;
  transition: all 0.2s ease;
}

.btn-buy:hover {
  background-color: #34d399;
}

.btn-sell {
  padding: 6px 12px;
  background-color: #fca5a5;
  color: #991b1b;
  border: 1px solid #fecaca;
  border-radius: 6px;
  font-size: 14px;
  transition: all 0.2s ease;
}

.btn-sell:hover {
  background-color: #f87171;
}

.add-stock-form {
  background-color: #f3f4f6;
  padding: 16px;
  border-radius: 10px;
  max-width: 400px;
}

.form-title {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 12px;
  color: #374151;
}

.form-fields {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-fields input {
  padding: 8px 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 14px;
}

.form-fields button {
  background-color: #3b82f6;
  color: white;
  padding: 8px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  transition: background 0.2s ease;
}

.form-fields button:hover {
  background-color: #2563eb;
}
</style>