<template>
  <div class="player-list">
    <h2 class="title">👥 플레이어 목록</h2>

    <div v-if="players.length === 0" class="empty">등록된 플레이어가 없습니다</div>

    <div class="player-card" v-for="player in players" :key="player.playerId">
      <div class="player-header">
        <span class="player-name">{{ player.playerId }}</span>
        <span class="player-money">{{ player.playerMoney?.toLocaleString() || 0 }} 원</span>
      </div>

      <div class="player-stocks">
        <template v-if="player.playerStocks && player.playerStocks.length">
          <div
            v-for="stock in player.playerStocks"
            :key="stock.stockName"
            class="stock"
          >
            <span class="stock-name">{{ stock.stockName }}</span>
            <span class="stock-quantity">{{ stock.stockQuantity }}주</span>
            <span class="stock-price">({{ stock.stockPrice.toLocaleString() }}원)</span>
          </div>
        </template>
        <div v-else class="text-gray-400 italic">보유 주식 없음</div>
      </div>

      <button
        v-if="currentPlayerId === player.playerId"
        @click="removePlayer(player.playerId)"
        class="remove-btn"
      >
        탈퇴
      </button>
      <button
        v-if="currentPlayerId === player.playerId"
        @click="openAddMoneyPrompt(player.playerId)"
        class="add-money-btn"
      >
        투자금 추가
      </button>
    </div>

    <!-- 새로운 플레이어 생성 폼 -->
    <div class="add-player-form">
      <h3 class="form-title">➕ 새로운 플레이어 생성</h3>
      <div class="form-fields">
        <input
          v-model="newPlayerId"
          placeholder="플레이어 이름"
        />
        <input
          v-model.number="initialMoney"
          type="number"
          placeholder="초기 투자금"
        />
        <button @click="createPlayer">생성하기</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const props = defineProps(['players', 'currentPlayerId'])
const emit = defineEmits(['player-created', 'refresh'])

const newPlayerId = ref('')
const initialMoney = ref(10000)

const removePlayer = async (playerId) => {
  const player = props.players.find(p => p.playerId === playerId)
  if (!player) return

  const ownedStocks = player.playerStocks?.filter(stock => stock.stockQuantity > 0) || []

  if (ownedStocks.length > 0) {
    alert(`보유한 주식이 남아 있으면 탈퇴할 수 없습니다.\n\n남아 있는 주식: ${ownedStocks.map(s => `${s.stockName} (${s.stockQuantity}주)`).join(', ')}`)
    return
  }

  if (confirm(`${playerId} 플레이어를 정말 탈퇴시키겠습니까?`)) {
    try {
      // DELETE 메서드는 그대로 유지 (변경 필요 없음)
      await axios.delete(`http://localhost:8080/api/players/${playerId}`)
      window.location.reload()
    } catch (error) {
      alert('플레이어 탈퇴 중 오류가 발생했습니다.')
    }
  }
}

const createPlayer = async () => {
  const name = newPlayerId.value.trim()
  const money = initialMoney.value
  
  if (!name) {
    return alert('플레이어 이름을 입력하세요')
  }
  
  if (money < 0) {
    return alert('초기 투자금은 0원 이상이어야 합니다')
  }
  
  try {
    // 플레이어 생성 - params에서 요청 본문으로 변경
    const response = await axios.post('http://localhost:8080/api/players', {
      id: name
    })
    
    // 사용자가 추가 자금을 입력한 경우에만 추가 자금 투입
    if (money > 0 && money !== 10000) { // 10000이 서버에서 기본으로 제공하는 금액이라면
      await axios.post(`http://localhost:8080/api/players/${name}/money`, {
        amount: money - 10000 // 차이만큼만 추가
      })
    }
    
    const message = money === 10000 ? 
      `${name} 플레이어가 기본 투자금으로 생성되었습니다!` :
      `${name} 플레이어가 ${money.toLocaleString()}원의 초기 투자금으로 생성되었습니다!`;
    
    alert(message)
    newPlayerId.value = ''
    initialMoney.value = 10000
    
    // 부모 컴포넌트에 이벤트 발생
    emit('player-created', name)
    emit('refresh')
  } catch (error) {
    if (error.response && error.response.status === 409) {
      alert('⚠️ 이미 존재하는 플레이어입니다!')
    } else {
      alert('⚠️ 플레이어 생성 중 오류가 발생했습니다.')
    }
  }
}

const openAddMoneyPrompt = async (playerId) => {
  const amountStr = prompt('얼마를 추가하시겠습니까?', '10000')
  const amount = parseInt(amountStr)
  if (isNaN(amount) || amount <= 0) {
    alert('올바른 금액을 입력해주세요.')
    return
  }

  try {
    // 자금 추가 - params에서 요청 본문으로 변경
    await axios.post(`http://localhost:8080/api/players/${playerId}/money`, {
      amount: amount
    })

    // ✅ player의 금액만 업데이트 (새로고침 없이)
    const target = props.players.find(p => p.playerId === playerId)
    if (target) {
      target.playerMoney += amount
    }

    alert(`₩${amount.toLocaleString()} 이(가) 추가되었습니다!`)
  } catch (error) {
    console.error('투자금 추가 중 오류:', error);
    if (error.response?.data?.message) {
      alert(`투자금 추가 실패: ${error.response.data.message}`);
    } else {
      alert('투자금 추가 중 오류가 발생했습니다.');
    }
  }
}
</script>

<style scoped>
.player-list {
  font-family: 'Pretendard', 'Noto Sans KR', sans-serif;
}

.title {
  font-size: 20px;
  font-weight: 600;
  margin-bottom: 16px;
  color: #1f2937;
}

.empty {
  color: #888;
  font-size: 14px;
  padding: 16px;
  text-align: center;
  border: 1px dashed #ccc;
  border-radius: 8px;
  background: #f9fafb;
}

.player-card {
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 16px;
  margin-bottom: 16px;
  background-color: #ffffff;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

.player-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 12px;
}

.player-name {
  font-size: 16px;
  font-weight: 600;
  color: #111827;
}

.player-money {
  font-size: 14px;
  font-weight: 500;
  color: #2563eb;
}

.player-stocks {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-bottom: 12px;
}

.stock {
  background-color: #e0f2fe;
  color: #0369a1;
  font-size: 13px;
  padding: 4px 8px;
  border-radius: 9999px;
}

.stock-price {
  margin-left: 4px;
  font-size: 12px;
  color: #6b7280;
}

.remove-btn {
  padding: 6px 12px;
  background-color: #fee2e2;
  color: #b91c1c;
  border: 1px solid #fca5a5;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.remove-btn:hover {
  background-color: #fecaca;
}

.add-money-btn {
  padding: 6px 12px;
  margin-left: 8px;
  background-color: #d1fae5;
  color: #047857;
  border: 1px solid #6ee7b7;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.add-money-btn:hover {
  background-color: #a7f3d0;
}

/* 새로운 플레이어 생성 폼 스타일 */
.add-player-form {
  background-color: #f3f4f6;
  padding: 16px;
  border-radius: 10px;
  margin-top: 24px;
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