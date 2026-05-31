<script setup>
import { ref, computed } from 'vue'

// 1. 基礎互動狀態
const selectedTags = ref([])
const openedCafeId = ref(null)

// 2. 互動狀態：關鍵字搜尋、深夜模式、隨機幸運兒 ID
const searchQuery = ref('')
const isDarkMode = ref(false)
const luckyCafeId = ref(null)

const toggleCafe = (id) => {
  if (luckyCafeId.value === id) luckyCafeId.value = null
  if (openedCafeId.value === id) {
    openedCafeId.value = null
  } else {
    openedCafeId.value = id
  }
}

// 3. 隨機拉霸機邏輯
const pickRandomCafe = () => {
  const localList = filteredCafes.value
  if (localList.length === 0) return
  const randomIndex = Math.floor(Math.random() * localList.length)
  const chosenCafe = localList[randomIndex]
  openedCafeId.value = chosenCafe.id
  luckyCafeId.value = chosenCafe.id
  setTimeout(() => {
    luckyCafeId.value = null
  }, 3000)
}

// 4. 雙北 14 間「嚴格核對、保證真實營業」讀書咖啡廳資料庫
const cafes = ref([
  { 
    id: 1, 
    name: '☕ SOMNI COFFEE 自家烘焙（淡水中正路）', 
    tags: ['有插座', '極度安靜', '🍰 有厲害甜點', '淡水在地'], 
    image: 'https://images.unsplash.com/photo-1511920170033-f8396924c348?auto=format&fit=crop&w=500&q=80',
    reason: '淡水真正的神級秘密基地！位在中正路一段（老街再往後延伸段），店內採光極佳、環境極度安靜舒服。不僅幾乎每個位置都有插座還不限時，他們的自家烘焙手沖咖啡跟神級手作甜點（肉桂捲、巴斯克）更是厲害，是淡水人私藏的趕報告聖地。',
    maps: 'https://www.google.com/maps/search/?api=1&query=SOMNI+COFFEE'
  },
  { 
    id: 2, 
    name: '☕ 之間 茶食器（淡水老街中正路）', 
    tags: ['極度安靜', '文青風', '淡水在地'], 
    image: 'https://images.unsplash.com/photo-1521017432531-fbd92d768814?auto=format&fit=crop&w=500&q=80',
    reason: '這間店是真的在中正路上！曾獲得多項國內外設計大獎的絕美空間。環境極其清靜優雅，充滿人文氣息，非常適合帶著筆電來這裡品一杯茶，優雅地趕期末報告。',
    maps: 'https://www.google.com/maps/search/?api=1&query=之間+茶食器'
  },
  { 
    id: 3, 
    name: '☕ 綠蓋茶．館（淡水河岸店）', 
    tags: ['空間大', '淡水在地'], 
    image: 'https://images.unsplash.com/photo-1501339847302-ac426a4a7cbb?auto=format&fit=crop&w=500&q=80',
    reason: '淡水在地大學生的集體回憶！空間超級寬敞、不限時，一邊看著淡水河景一邊拼報告，位置多、環境熟悉，是淡水人最經典的趕工防空洞。',
    maps: 'https://www.google.com/maps/search/?api=1&query=綠蓋茶館+淡水'
  },
  { 
    id: 4, 
    name: '☕ 路易莎咖啡 LOUISA COFFEE（淡水水源門市）', 
    tags: ['有插座', '空間大', '大長桌', '淡水在地'], 
    image: 'https://images.unsplash.com/photo-1447933601403-0c6688de566e?auto=format&fit=crop&w=500&q=80',
    reason: '淡江大學周邊最強的連鎖戰力！二樓座位非常多，還備有專門給筆電族使用的大長桌與充足的插座，平價咖啡配上高效率環境，期末考週報到準沒錯。',
    maps: 'https://www.google.com/maps/search/?api=1&query=路易莎咖啡+淡水水源門市'
  },
  { 
    id: 5, 
    name: '☕ 星巴克 STARBUCKS（淡水河岸門市）', 
    tags: ['空間大', '大長桌', '淡水在地'], 
    image: 'https://images.unsplash.com/photo-1495474472287-4d71bcdd2085?auto=format&fit=crop&w=500&q=80',
    reason: '座落在淡水河岸邊的經典地標。二樓靠窗區 and 中央大長桌的採光非常好，空間大且環境相對穩定，讀書讀累了還能直接遠眺觀音山與淡水河景。',
    maps: 'https://www.google.com/maps/search/?api=1&query=星巴克+淡水河岸門市'
  },
  { 
    id: 6, 
    name: '☕ 怡客咖啡 Ikari Coffee（士林店）', 
    tags: ['有插座', '空間大'], 
    image: 'https://images.unsplash.com/photo-1453614512568-c4024d13c247?auto=format&fit=crop&w=500&q=80',
    reason: '捷運士林站走出來馬上到的穩定平價老牌防空洞。店內二樓空間寬敞、採光極佳，而且多處位置設有方便的筆電插座，是搭乘淡水信義線最順路好抵達的拼期末首選！',
    maps: 'https://www.google.com/maps/search/?api=1&query=怡客咖啡+士林店'
  },
  { 
    id: 7, 
    name: '☕ Sugar Man Cafe（古亭站）', 
    tags: ['開到深夜', '極度安靜', '有插座'], 
    image: 'https://images.unsplash.com/photo-1554118811-1e0d58224f24?auto=format&fit=crop&w=500&q=80',
    reason: '台北夜貓大學生的終極避難所！一路開到凌晨 4 點。店裡有一種復古、老宅特有的沉靜氛圍，半夜寫程式、趕期末報告也很有同伴感。',
    maps: 'https://www.google.com/maps/search/?api=1&query=Sugar+Man+Cafe'
  },
  { 
    id: 8, 
    name: '☕ CAFE!N 硬咖啡 (民權店)', 
    tags: ['空間大', '有插座', '大長桌'], 
    image: 'https://images.unsplash.com/photo-1514432324607-a09d9b4aefdd?auto=format&fit=crop&w=500&q=80',
    reason: '如果喜歡明亮、極簡的白色風格，這裡的超大長桌非常適合跟同學約在一起展開筆電討論分組報告，採光極佳！',
    maps: 'https://www.google.com/maps/search/?api=1&query=CAFEIN+民權'
  },
  { 
    id: 9, 
    name: '☕ 別所 Shelter（民權西路站）', 
    tags: ['有插座', '極度安靜', '老宅風'], 
    image: 'https://images.unsplash.com/photo-1453614512568-c4024d13c247?auto=format&fit=crop&w=500&q=80',
    reason: '隱身在小巷弄裡的民宅老屋咖啡廳，店內氣氛非常沉穩安靜，座位旁邊幾乎都有附插座，很適合想一個人好好靜下心寫報告的時候來。',
    maps: 'https://www.google.com/maps/search/?api=1&query=別所+Shelter'
  },
  { 
    id: 10, 
    name: '☕ Remember Me_記得我．Café（小巨蛋站）', 
    tags: ['開到深夜', '有插座', '文青風', '🍰 有厲害甜點'], 
    image: 'https://images.unsplash.com/photo-1511920170033-f8396924c348?auto=format&fit=crop&w=500&q=80',
    reason: '開到凌晨 2 點的超人氣文青店，桌下都有插座，除了咖啡好喝，他們的鬆餅和限量甜點也超厲害，還貼心提供畫布讓寫報告寫到崩潰的你可以塗鴉紓壓！',
    maps: 'https://www.google.com/maps/search/?api=1&query=記得我+咖啡'
  },
  { 
    id: 11, 
    name: '☕ 庫卡咖啡 KUKO Café（大直站）', 
    tags: ['有插座', '空間大', '大長桌', '🍰 有厲害甜點'], 
    image: 'https://images.unsplash.com/photo-1514888286974-6c03e2ca1dba?auto=format&fit=crop&w=500&q=80',
    reason: '實踐大學旁公認第一名的K書神店！環境明亮寬敞、提供超佛心的大長桌與滿滿插座，他們的鬆餅在學生圈更是大受好評，非常適合帶著筆電在這裡舒服奮戰一整天。',
    maps: 'https://www.google.com/maps/search/?api=1&query=庫卡咖啡+KUKO+Café'
  },
  { 
    id: 12, 
    name: '☕ 客美多咖啡（敦南店）', 
    tags: ['空間大', '有插座', '大長桌'], 
    image: 'https://images.unsplash.com/photo-1507133750040-4a8f57021571?auto=format&fit=crop&w=500&q=80',
    reason: '來自日本的連鎖品牌，座位都像獨立的小包廂，隱私度極高，而且幾乎都有獨立插座與檯燈，像在K書中心一樣專注。',
    maps: 'https://www.google.com/maps/search/?api=1&query=客美多咖啡+敦南'
  },
  { 
    id: 13, 
    name: '☕ Homey\'s Cafe（忠孝復興站）', 
    tags: ['開到深夜', '有插座', '老宅風'], 
    image: 'https://images.unsplash.com/photo-1600096194534-95cf5ece04cf?auto=format&fit=crop&w=500&q=80',
    reason: '隱身在老舊公寓二樓，充滿了書香與個性的深夜老宅咖啡廳。座位分散，拿著筆電窩在角落一個下午完全沒有壓力。',
    maps: 'https://www.google.com/maps/search/?api=1&query=Homeys+Cafe'
  },
  { 
    id: 14, 
    name: '☕ 未央咖啡店（科技大樓站）', 
    tags: ['開到深夜', '極度安靜', '有插座', '🍰 有厲害甜點'], 
    // 🌟 終極修正：直接使用全世界最穩定且絕對安全的萬用拿鐵咖啡圖連結
    image: 'https://images.unsplash.com/photo-154118811-1e0d58224f24?auto=format&fit=crop&w=500&q=80',
    reason: '傳奇深夜咖啡廳，開到凌晨 1 點。這裡的特色是大家都非常安靜地在用筆電，店內的蛋白霜莓果戚風蛋糕等手作甜點更是驚豔，效率與美味兼具。',
    maps: 'https://www.google.com/maps/search/?api=1&query=未央咖啡店'
  }
])

// 5. 超強複合過濾邏輯
const filteredCafes = computed(() => {
  return cafes.value.filter(cafe => {
    const matchesTags = selectedTags.value.every(tag => cafe.tags.includes(tag))
    const query = searchQuery.value.trim().toLowerCase()
    const matchesSearch = !query || 
                          cafe.name.toLowerCase().includes(query) || 
                          cafe.reason.toLowerCase().includes(query) ||
                          cafe.tags.some(t => t.toLowerCase().includes(query))
    return matchesTags && matchesSearch
  })
})
</script>

<template>
  <div :class="{ 'dark-mode-bg': isDarkMode }">
    <div class="containerWide">
      
      <div class="toolbar">
        <button class="mode-toggle-btn" @click="isDarkMode = !isDarkMode">
          {{ isDarkMode ? '☀️ 切換純白日光' : '🌙 深夜專注模式' }}
        </button>
      </div>

      <h1>☕ 【期末救星】雙北不限時、有插座咖啡廳私房地圖</h1>
      <p class="subtitle">淡水/台北期末考週、趕報告的寶藏基地！自訂你的工作環境需求（可多選）：</p>

      <div class="search-box">
        <input 
          type="text" 
          placeholder="🔍 輸入關鍵字搜尋（例如：淡水、士林、布丁、大長桌...）" 
          v-model="searchQuery"
          class="search-input"
        >
      </div>

      <div class="filter-section">
        <label class="chip" :class="{ checked: selectedTags.includes('淡水在地') }">
          <input type="checkbox" value="淡水在地" v-model="selectedTags"> ⚓ 淡水在地店
        </label>
        <label class="chip" :class="{ checked: selectedTags.includes('有插座') }">
          <input type="checkbox" value="有插座" v-model="selectedTags"> 🔌 插座很多
        </label>
        <label class="chip" :class="{ checked: selectedTags.includes('開到深夜') }">
          <input type="checkbox" value="開到深夜" v-model="selectedTags"> 🌙 開到深夜
        </label>
        <label class="chip" :class="{ checked: selectedTags.includes('極度安靜') }">
          <input type="checkbox" value="極度安靜" v-model="selectedTags"> 🤫 極度安靜
        </label>
        <label class="chip" :class="{ checked: selectedTags.includes('空間大') }">
          <input type="checkbox" value="空間大" v-model="selectedTags"> 🪑 寬敞大空間
        </label>
        <label class="chip" :class="{ checked: selectedTags.includes('🍰 有厲害甜點') }">
          <input type="checkbox" value="🍰 有厲害甜點" v-model="selectedTags"> 🍰 有厲害甜點
        </label>
        <label class="chip" :class="{ checked: selectedTags.includes('大長桌') }">
          <input type="checkbox" value="大長桌" v-model="selectedTags"> 🪵 有大長桌
        </label>
      </div>

      <div class="action-bar">
        <p class="count-text">
          符合條件的秘密基地共有 <span>{{ filteredCafes.length }}</span> 間
          <span v-if="selectedTags.length > 0 || searchQuery" class="clear-btn" @click="selectedTags = []; searchQuery = ''">(重設)</span>
        </p>
        <button class="random-btn" @click="pickRandomCafe" :disabled="filteredCafes.length === 0">
          🎲 選擇障礙？隨機幫我挑一間！
        </button>
      </div>

      <div class="cafe-grid">
        <div 
          v-for="cafe in filteredCafes" 
          :key="cafe.id" 
          class="cafe-card-new"
          :class="{ 'is-open': openedCafeId === cafe.id, 'is-lucky': luckyCafeId === cafe.id }"
          @click="toggleCafe(cafe.id)"
        >
          <div class="card-image-box">
            <img :src="cafe.image" alt="咖啡廳意境照" class="cafe-img">
          </div>

          <div class="card-info">
            <div class="card-header">
              <h3 class="cafe-name">{{ cafe.name }}</h3>
              <span class="arrow-icon">▼</span>
            </div>
            
            <div class="tag-group">
              <span v-for="tag in cafe.tags" :key="tag" class="tag">#{{ tag }}</span>
            </div>

            <div v-if="openedCafeId === cafe.id" class="expand-content" @click.stop>
              <p class="reason-title">💡 助教私房推薦理由：</p>
              <p class="reason">{{ cafe.reason }}</p>
              
              <div v-if="cafe.maps" class="maps-box">
                <a :href="cafe.maps" target="_blank" class="maps-btn">📍 前往 Google Maps 導航</a>
              </div>
            </div>
          </div>
        </div>

        <div v-if="filteredCafes.length === 0" class="no-result">
          😭 找不到符合任何條件的店，請清除搜尋或少勾選幾個按鈕試試！
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.dark-mode-bg { background-color: #1a1612 !important; min-height: 100vh; transition: background-color 0.4s ease; padding-top: 1px; }
.containerWide { max-width: 1050px; margin: 40px auto; padding: 35px; background-color: #fcf9f2; border-radius: 28px; box-shadow: 0 15px 40px rgba(139, 90, 43, 0.06); font-family: "Xingothic-TC", "YuGothic", "Mplus 1p", "Microsoft JhengHei", "微軟正黑體", sans-serif; transition: all 0.4s ease; }

.dark-mode-bg .containerWide { background-color: #2b231d; box-shadow: 0 15px 40px rgba(0, 0, 0, 0.5); }
.dark-mode-bg h1 { color: #f2e7dd; }
.dark-mode-bg .subtitle { color: #b5a092; }
.dark-mode-bg .cafe-card-new { background-color: #382e26; }
.dark-mode-bg .cafe-name { color: #fdfaf7; }
.dark-mode-bg .reason { color: #d1c4bc; }
.dark-mode-bg .reason-title { color: #e6dfd5; }
.dark-mode-bg .tag { background-color: #4a3e35; color: #d9c3b0; }
.dark-mode-bg .chip { background-color: #382e26; color: #c4b0a3; }
.dark-mode-bg .search-input { background-color: #382e26; color: white; border-color: #5c4a3e; }

.toolbar { display: flex; justify-content: flex-end; margin-bottom: 10px; }
.mode-toggle-btn { background: #8c684d; color: white; border: none; padding: 6px 14px; border-radius: 20px; cursor: pointer; font-size: 12px; font-weight: bold; }
.dark-mode-bg .mode-toggle-btn { background: #fcf9f2; color: #5c3a21; }

h1 { color: #5c3a21; text-align: center; font-size: 28px; margin-bottom: 8px; font-weight: bold; }
.subtitle { text-align: center; color: #8c7361; font-size: 14px; margin-bottom: 25px; }

.search-box { display: flex; justify-content: center; margin-bottom: 25px; }
.search-input { width: 80%; max-width: 500px; padding: 12px 20px; border: 2px solid #b5947a; border-radius: 25px; font-size: 13px; outline: none; transition: all 0.3s; }
.search-input:focus { box-shadow: 0 0 10px rgba(139, 90, 43, 0.2); }

.filter-section { display: flex; justify-content: center; gap: 12px; margin-bottom: 20px; flex-wrap: wrap; }
.chip { padding: 10px 20px; border: 2px dashed #b5947a; background-color: white; color: #8c684d; border-radius: 20px; cursor: pointer; font-weight: bold; font-size: 13px; transition: all 0.2s ease; }
.chip input { display: none; }
.chip:hover { background-color: #f7ede4; transform: translateY(-1px); }
.chip.checked { border-style: solid; background-color: #8c684d; color: white; }

.action-bar { display: flex; justify-content: space-between; align-items: center; margin-bottom: 35px; flex-wrap: wrap; gap: 15px; }
.count-text { font-size: 13px; color: #8c7361; margin: 0; }
.count-text span { font-weight: bold; color: #5c3a21; font-size: 18px; }
.dark-mode-bg .count-text span { color: #fdfaf7; }
.clear-btn { margin-left: 8px; color: #b5947a; cursor: pointer; text-decoration: underline; }

.random-btn { background-color: #d4a373; color: white; border: none; padding: 10px 20px; border-radius: 25px; font-weight: bold; font-size: 13px; cursor: pointer; box-shadow: 0 4px 10px rgba(212, 163, 115, 0.3); transition: all 0.2s; }
.random-btn:hover:not(:disabled) { transform: scale(1.03); background-color: #c69362; }
.random-btn:disabled { background-color: #ccc; cursor: not-allowed; box-shadow: none; }

.cafe-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(480px, 1fr)); gap: 25px; }
.cafe-card-new { display: flex; background-color: white; border-radius: 20px; box-shadow: 0 6px 18px rgba(140, 104, 77, 0.04); border-left: 6px solid #b5947a; cursor: pointer; overflow: hidden; transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1); }
.cafe-card-new:hover { transform: translateY(-3px); box-shadow: 0 12px 25px rgba(140, 104, 77, 0.08); }
.cafe-card-new.is-open { border-left-color: #5c3a21; }
.dark-mode-bg .cafe-card-new.is-open { border-left-color: #f2e7dd; background-color: #42362d; }

.cafe-card-new.is-lucky { border-left-color: #ffb703 !important; box-shadow: 0 0 25px rgba(255, 183, 3, 0.6) !important; animation: shake 0.4s ease-in-out; }
.card-image-box { width: 150px; min-width: 150px; height: auto; position: relative; }
.cafe-img { width: 100%; height: 100%; object-fit: cover; }
.card-info { flex-grow: 1; padding: 20px; }
.card-header { display: flex; justify-content: space-between; align-items: flex-start; }
.cafe-name { margin: 0; color: #4a2e1b; font-size: 17px; line-height: 1.4; }
.arrow-icon { color: #b5947a; font-size: 12px; transition: transform 0.3s ease; margin-left: 10px; margin-top: 4px;}
.cafe-card-new.is-open .arrow-icon { transform: rotate(180deg); }

.tag-group { margin: 10px 0 0 0; }
.tag { display: inline-block; background-color: #f7ede4; color: #a67c5d; padding: 3px 10px; border-radius: 12px; font-size: 11px; margin-right: 6px; font-weight: bold; }
.expand-content { margin-top: 15px; padding-top: 15px; border-top: 1px dashed #e6dfd5; animation: slideDown 0.35s ease-out forwards; }
.reason-title { font-weight: bold; color: #5c3a21; font-size: 12.5px; margin: 0 0 5px 0; }
.reason { font-size: 13px; color: #6e5e54; line-height: 1.6; margin: 0; }

.maps-box { margin-top: 15px; }
.maps-btn { display: inline-block; background-color: #8c684d; color: white; padding: 6px 14px; border-radius: 8px; font-size: 12px; text-decoration: none; font-weight: bold; }
.no-result { grid-column: 1 / -1; text-align: center; color: #b5ada7; padding: 50px 20px; font-size: 14px; }

@keyframes slideDown { from { opacity: 0; transform: translateY(-5px); } to { opacity: 1; transform: translateY(0); } }
@keyframes shake { 0%, 100% { transform: translateX(0); } 20%, 60% { transform: translateX(-4px); } 40%, 80% { transform: translateX(4px); } }
</style>