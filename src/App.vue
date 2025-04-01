<template>
  <main class="container">
    <!-- 상단: 타이틀 + 플레이어 입력 -->
    <div class="top-bar">
      <h1 class="title">
        📈 Skala 주식 거래
        <span v-if="playerId" class="connected">- 👤 {{ playerId }}님 접속 중</span>
      </h1>
      <div v-if="!playerId" class="player-input">
        <input
          v-model="inputPlayerId"
          placeholder="플레이어 이름 입력"
        />
        <button @click="login">접속하기</button>
      </div>
      <div v-else class="player-input">
        <button @click="logout" class="logout-btn">로그아웃</button>
      </div>
    </div>

    <!-- 본문: 좌우 레이아웃 -->
    <div class="main-section">
      <div class="left">
        <StockList
          :stocks="stocks"
          @buy="handleBuy"
          @sell="handleSell"
          @refresh="loadData"
        />
      </div>
      <div class="right">
        <PlayerList 
          :players="players" 
          :current-player-id="playerId" 
          :key="JSON.stringify(players)" 
          @player-created="handlePlayerCreated"
          @refresh="loadData"
        />
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'
import StockList from './components/StockList.vue'
import PlayerList from './components/PlayerList.vue'

const tab = ref('home')
const tabClass = (target) => (tab.value === target ? 'active' : '')

const stocks = ref([])
const players = ref([])
const playerId = ref('')
const inputPlayerId = ref('')
const newStockName = ref('')
const newStockPrice = ref('')

const loadData = async () => {
  stocks.value = (await axios.get('http://localhost:8080/api/stocks')).data
  players.value = (await axios.get('http://localhost:8080/api/players')).data
}

const handleBuy = async (player, stock, quantity) => {
  if (!playerId.value) return alert('먼저 플레이어 이름을 입력하세요')
  try {
    await axios.post(`http://localhost:8080/api/players/${playerId.value}/buy`, null, {
      params: { stockName: stock, quantity }
    })
    alert('매수 성공!')
    await loadData()
  } catch (error) {
    if (error.response?.data) {
      alert(`매수 실패: ${error.response.data}`)
    } else if (error.response?.status === 400) {
      alert('매수 실패: 자금이 부족합니다.')
    } else {
      alert('매수 중 오류가 발생했습니다.')
    }
  }
}

const handleSell = async (stock, quantity) => {
  if (!playerId.value) return alert('먼저 플레이어 이름을 입력하세요')
  try {
    await axios.post(`http://localhost:8080/api/players/${playerId.value}/sell`, null, {
      params: { stockName: stock, quantity }
    })
    alert('매도 성공!')
    await loadData()
  } catch (error) {
    if (error.response?.data) {
      alert(`매도 실패: ${error.response.data}`)
    } else if (error.response?.status === 400) {
      alert('매도 실패: 보유한 주식이 부족합니다.')
    } else {
      alert('매도 중 오류가 발생했습니다.')
    }
  }
}

const login = async () => {
  const name = inputPlayerId.value.trim()
  if (!name) return alert('플레이어 이름을 입력하세요')

  try {
    await axios.get(`http://localhost:8080/api/players/${name}`)
    playerId.value = name
    alert(`${playerId.value}님 접속 완료!`)
    await loadData()
    inputPlayerId.value = ''
  } catch (error) {
    if (error.response && error.response.status === 404) {
      alert('⚠️ 등록되지 않은 플레이어입니다. 먼저 생성해주세요!')
    } else {
      alert('⚠️ 접속 중 오류가 발생했습니다.')
    }
  }
}

const logout = () => {
  if (confirm(`${playerId.value}님, 정말 로그아웃 하시겠습니까?`)) {
    playerId.value = ''
    alert('로그아웃 되었습니다.')
  }
}

const handlePlayerCreated = (id) => {
  // 플레이어 생성 후 자동 로그인 (옵션)
  if (confirm(`${id} 플레이어로 자동 로그인하시겠습니까?`)) {
    playerId.value = id
    alert(`${id}님 접속 완료!`)
  }
}

const createPlayer = async () => {
  const name = inputPlayerId.value.trim()
  if (!name) return alert('플레이어 이름을 입력하세요')
  
  try {
    await axios.post('http://localhost:8080/api/players', null, {
      params: { id: name }
    })
    playerId.value = name
    alert(`${name} 플레이어가 생성되었습니다!`)
    await loadData()
    inputPlayerId.value = ''
  } catch (error) {
    if (error.response && error.response.status === 409) {
      alert('⚠️ 이미 존재하는 플레이어입니다!')
    } else {
      alert('⚠️ 플레이어 생성 중 오류가 발생했습니다.')
    }
  }
}

// const addStock = async () => {
//   const name = newStockName.value.trim()
//   const price = parseInt(newStockPrice.value)

//   if (!name) return alert('주식 이름을 입력하세요')
//   if (!price || price <= 0) return alert('유효한 가격을 입력하세요')

//   try {
//     await axios.post('http://localhost:8080/api/stocks', null, {
//       params: { name, price }
//     })
//     alert(`${name} 주식이 추가되었습니다!`)
//     newStockName.value = ''
//     newStockPrice.value = ''
//     await loadData()
//   } catch (error) {
//     if (error.response && error.response.status === 409) {
//       alert('⚠️ 이미 존재하는 주식입니다!')
//     } else {
//       alert('⚠️ 주식 추가 중 오류가 발생했습니다.')
//     }
//   }
// }

onMounted(loadData)
</script>

<style scoped>
.container {
  max-width: 1000px;
  margin: auto;
  padding: 20px;
  font-family: 'Pretendard', 'Noto Sans KR', sans-serif;
  color: #1f2937;
}

.connected {
  font-size: 16px;
  font-weight: normal;
  color: #374151;
  margin-left: 10px;
}

.top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
  flex-wrap: wrap;
}

.top-bar .title {
  font-size: 24px;
  font-weight: 600;
  margin-bottom: 10px;
  color: #1f2937;
}

.player-input {
  display: flex;
  align-items: center;
}

.player-input input {
  padding: 8px 12px;
  font-size: 14px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  outline: none;
  transition: border-color 0.2s;
}

.player-input input:focus {
  border-color: #3b82f6;
}

.player-input button {
  padding: 8px 14px;
  margin-left: 8px;
  background-color: #dbeafe;
  color: #1d4ed8;
  border: 1px solid #93c5fd;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.player-input button:hover {
  background-color: #bfdbfe;
}

.logout-btn {
  background-color: #fee2e2 !important;
  color: #b91c1c !important;
  border: 1px solid #fca5a5 !important;
}

.logout-btn:hover {
  background-color: #fecaca !important;
}

.main-section {
  display: flex;
  gap: 20px;
  flex-wrap: wrap;
}

.left,
.right {
  flex: 1;
  min-width: 300px;
  border: 1px solid #e5e7eb;
  padding: 16px;
  border-radius: 12px;
  background-color: #ffffff;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}
</style>