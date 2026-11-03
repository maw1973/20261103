<script setup>
import { computed, ref, watch } from 'vue'
import { Wrench, Search, CalendarDays, Calculator, Laptop, Smartphone, Monitor, Tablet, ArrowRight, CheckCircle2, ShieldCheck, HeartHandshake, MapPin, Phone, Menu, X, ArrowLeft, ClipboardList, Sparkles, Brush, Info, Plus, Minus, ExternalLink, Cpu } from 'lucide-vue-next'

const page = ref('home')
const menuOpen = ref(false)
const toast = ref('')
function readSavedList(key) {
  try { const data = JSON.parse(localStorage.getItem(key) || '[]'); return Array.isArray(data) ? data : [] }
  catch { return [] }
}
const notices = ref(readSavedList('caretech-notices'))
const reservations = ref(readSavedList('caretech-reservations'))
const activeFaq = ref(-1)
const serviceFilter = ref('全部')
const inquiryId = ref('')
const inquiryPhone = ref('')
const inquiryResult = ref(null)
// 以下均為網站展示用模擬價格，非正式報價或門市承諾。
const priceCatalog = {
  '筆電': [
    ['螢幕閃爍（基本排線處理）',1000,false,false],['螢幕顯示異常檢測',500,false,false],['更換一般液晶螢幕',4500,true,false],
    ['鍵盤故障更換',2000,true,false],['電池更換',2000,true,false],['無法開機檢測',500,false,false],
    ['散熱風扇清潔',800,false,false],['風扇故障更換',1500,true,false],['Windows 系統重灌',1000,false,false],
    ['SSD 安裝工資',500,false,true],['記憶體升級工資',300,false,true]
  ],
  '桌機': [
    ['電腦無法開機檢測',300,false,false],['Windows 系統重灌',800,false,false],['電腦中毒處理',800,true,false],
    ['電腦運作緩慢優化',500,false,false],['電源供應器更換工資',500,false,true],['SSD 更換工資',400,false,true],
    ['RAM 記憶體安裝工資',200,false,true],['顯示卡安裝工資',300,false,true],['CPU 散熱膏更換',500,false,false],
    ['主機內部除塵',500,false,false],['電腦組裝工資',1000,false,true]
  ],
  '手機': [
    ['手機基本故障檢測',300,false,false],['Android 電池更換',1000,true,false],['iPhone 電池更換',1500,true,false],
    ['Android 螢幕更換',2000,true,false],['iPhone 螢幕更換',3000,true,false],['充電孔清潔',300,false,false],
    ['充電孔模組更換',1200,true,false],['喇叭故障維修',800,true,false],['手機資料轉移',500,true,false]
  ],
  '平板': [
    ['平板基本檢測',300,false,false],['電池更換',1500,true,false],['觸控面板更換',2000,true,false],
    ['液晶螢幕更換',3000,true,false],['充電孔故障',1000,true,false],['系統異常處理',800,false,false],
    ['資料備份及轉移',500,true,false]
  ],
  '清潔／軟體': [
    ['筆電外部清潔',300,false,false],['筆電內部除塵',800,false,false],['筆電散熱膏更新',800,false,false],
    ['桌機基本除塵',500,false,false],['桌機深度清潔',1000,false,false],['Windows 系統優化',500,false,false],
    ['驅動程式安裝',300,false,false],['印表機連線設定',300,false,false],['資料備份（一般容量）',500,true,false]
  ]
}
const estimatedDevice = ref('筆電')
const selectedItems = ref([])
const availablePrices = computed(() => priceCatalog[estimatedDevice.value].map(([name,price,starting,labor]) => ({name,price,starting,labor})))
const selectedPrices = computed(() => availablePrices.value.filter(item => selectedItems.value.includes(item.name)))
const estimateTotal = computed(() => selectedPrices.value.reduce((sum,item) => sum + item.price,0))
const estimateIsMinimum = computed(() => selectedPrices.value.some(item => item.starting))
const nt = n => 'NT$ ' + n.toLocaleString('zh-TW')
watch(estimatedDevice, () => { selectedItems.value = [] })
function togglePrice(name) {
  selectedItems.value = selectedItems.value.includes(name) ? selectedItems.value.filter(x=>x!==name) : [...selectedItems.value,name]
}
function copyEstimate() {
  if (!selectedPrices.value.length) return notify('請先勾選至少一項維修服務')
  const lines = ['永茂3C｜模擬維修費用試算','設備：'+estimatedDevice.value,
    ...selectedPrices.value.map(x => `${x.name}：${nt(x.price)}${x.starting?' 起':''}${x.labor?'（不含零件）':''}`),
    `模擬${estimateIsMinimum.value?'起始合計':'合計'}：${nt(estimateTotal.value)}${estimateIsMinimum.value?' 起':''}`,
    '非正式報價；可能有重疊工項，是否併計與最終費用以門市檢測報價為準。']
  navigator.clipboard?.writeText(lines.join('\n')).then(()=>notify('已複製試算摘要')).catch(()=>notify('無法自動複製，請手動記錄試算結果'))
}
const serviceCatalog = [
  { id:'laptop', name:'筆電維修', icon:Laptop, tag:'筆電', brief:'無法開機、螢幕、鍵盤、電池與散熱處理', tone:'peach' },
  { id:'phone', name:'手機維修', icon:Smartphone, tag:'手機', brief:'螢幕、充電、電池與進水異常檢測', tone:'mint' },
  { id:'desktop', name:'桌機維修', icon:Monitor, tag:'桌機', brief:'主機板、系統異常、零件升級與組裝', tone:'blue' },
  { id:'tablet', name:'平板維修', icon:Tablet, tag:'平板', brief:'觸控、電池、顯示及充電問題', tone:'lavender' },
  { id:'clean', name:'清潔保養', icon:Brush, tag:'其他', brief:'風扇清潔、散熱檢查與保養建議', tone:'lemon' },
  { id:'software', name:'軟體支援', icon:Cpu, tag:'其他', brief:'系統安裝、軟體問題與資料處理諮詢', tone:'mint' }
]
const filteredServices = computed(() => serviceFilter.value === '全部' ? serviceCatalog : serviceCatalog.filter(s => s.tag === serviceFilter.value))
const submitStep = ref(1)
const accepted = ref(false)
const form = ref({device:'筆電',issue:'',name:'',phone:'',contact:'電話',note:''})
const book = ref({name:'',phone:'',date:'',time:'14:00',device:'筆電',note:''})
const questions = [
  ['沒有維修單號，可以查進度嗎？','此展示版查詢需要在本頁登記取得的單號及電話。正式版可以增加簡訊驗證或聯絡門市協助。'],
  ['線上費用試算是正式報價嗎？','不是。本站金額是規劃展示用模擬價目表，不是永茂3C正式收費。實際費用依品牌型號、零件與門市檢測後報價為準。'],
  ['送修前要先備份資料嗎？','建議送修前備份重要資料，並與門市確認資料保護、拆機與維修同意事項。'],
  ['線上預約後就有保留時段嗎？','此展示版的預約僅儲存在目前瀏覽器，尚未連接門市排程。正式版需後端確認時段才能成立。']
]
function navigate(next) { page.value = next; menuOpen.value = false; activeFaq.value = -1; window.scrollTo({top:0,behavior:'smooth'}) }
function notify(message) { toast.value = message; window.setTimeout(()=> { if(toast.value === message) toast.value = '' }, 3500) }
function newTicket() {
  if (!form.value.issue.trim() || !form.value.name.trim() || !/^09\d{8}$/.test(form.value.phone)) { notify('請確認故障說明、姓名與 10 碼手機號碼'); return }
  if (!accepted.value) { notify('請先閱讀並同意送修注意事項'); return }
  const id = 'DEMO-' + Date.now().toString().slice(-8)
  notices.value.unshift({...form.value,id,status:'已建立展示登記',date:new Date().toLocaleDateString('zh-TW')})
  localStorage.setItem('caretech-notices', JSON.stringify(notices.value))
  inquiryId.value = id; inquiryPhone.value = form.value.phone
  submitStep.value = 3
}
function lookup() {
  const ticket = notices.value.find(t => t.id.toUpperCase() === inquiryId.value.trim().toUpperCase() && t.phone === inquiryPhone.value.trim())
  inquiryResult.value = ticket || false
}
function submitBooking() {
  if(!book.value.name.trim() || !/^09\d{8}$/.test(book.value.phone) || !book.value.date || !book.value.time) { notify('請填寫姓名、手機號碼、日期與時間'); return }
  if (book.value.date < new Date().toLocaleDateString('en-CA')) { notify('請選擇今天或之後的日期'); return }
  const id = 'BOOK-' + Date.now().toString().slice(-7)
  reservations.value.unshift({...book.value,id})
  localStorage.setItem('caretech-reservations',JSON.stringify(reservations.value))
  notify(`已儲存展示預約 ${id}，尚未送達門市`)
}
function resetTicket(){ form.value={device:'筆電',issue:'',name:'',phone:'',contact:'電話',note:''};accepted.value=false;submitStep.value=1 }

</script>

<template>
  <div class="app-shell">
    <div class="topbar"><span><ShieldCheck :size="17"/> 有溫度的維修服務</span><span>太平坪林門市　<a href="tel:0423924505">(04) 2392-4505</a></span></div>
    <header class="header"><div class="header-inner">
      <button class="brand" @click="navigate('home')" aria-label="回到首頁"><span class="brand-icon"><Wrench :size="25" :stroke-width="2.5"/></span><span><b>永茂3C</b><small>讓科技恢復好心情</small></span></button>
      <nav class="nav" :class="{opened:menuOpen}" aria-label="主選單">
        <button :class="{active:page==='home'}" @click="navigate('home')">首頁</button><button :class="{active:page==='services'}" @click="navigate('services')">維修服務</button><button :class="{active:page==='booking'}" @click="navigate('booking')">線上預約</button><button :class="{active:page==='estimate'}" @click="navigate('estimate')">費用評估</button><button :class="{active:page==='faq'}" @click="navigate('faq')">常見問題</button>
        <button class="nav-mobile-cta" @click="navigate('register')">我要送修 <ArrowRight :size="18"/></button>
      </nav>
      <button class="header-cta" @click="navigate('register')">我要送修 <ArrowRight :size="18"/></button>
      <button class="menu-toggle" @click="menuOpen=!menuOpen" :aria-expanded="menuOpen" aria-label="開啟或關閉選單"><X v-if="menuOpen"/><Menu v-else/></button>
    </div></header>

    <main>
      <template v-if="page==='home'">
        <section class="hero"><div class="container hero-grid"><div class="hero-copy"><span class="eyebrow"><span class="pulse-dot"></span> 在地服務 · 用心處理每個問題</span><h1>電腦出狀況？<br/><em>交給我們，安心就好。</em></h1><p class="hero-desc">從筆電、手機到桌上型電腦，維修不該是一件複雜的事。清楚說明、簡單登記，讓每一步都有方向。</p><div class="hero-buttons"><button class="btn btn-primary" @click="navigate('register')">立即線上送修 <ArrowRight :size="20"/></button><button class="btn btn-outline" @click="navigate('lookup')"><Search :size="20"/> 查詢維修進度</button></div><div class="hero-points"><span><CheckCircle2 :size="18"/> 流程清楚</span><span><CheckCircle2 :size="18"/> 專人說明</span><span><CheckCircle2 :size="18"/> 多種設備</span></div></div>
          <div class="hero-art" aria-label="維修設備插圖"><div class="art-glow"></div><div class="big-device"><div class="device-display"><span class="device-smile">✦</span><span>Hi, 我們來幫忙！</span><div class="device-status"><CheckCircle2 :size="16"/> 用心檢查中</div></div><div class="device-foot"></div></div><div class="float-card float-top"><span class="mini-icon"><HeartHandshake :size="24"/></span><div><b>安心維修</b><small>每個細節都用心</small></div></div><div class="float-card float-bottom"><span class="mini-icon orange"><Sparkles :size="24"/></span><div><b>更簡單的流程</b><small>線上就能先登記</small></div></div><div class="sparkle one">✳</div><div class="sparkle two">✦</div></div>
        </div></section>
        <section class="quick-section container"><div class="section-heading"><div><span class="section-kicker">QUICK ACCESS</span><h2>今天想要我們幫什麼忙？</h2><p>四個常用入口，快速找到你需要的服務。</p></div></div><div class="quick-grid">
          <button class="quick-card q-peach" @click="navigate('register')"><span class="quick-icon"><ClipboardList/></span><h3>線上送修</h3><p>描述問題，建立送修資料</p><span class="quick-arrow"><ArrowRight/></span></button>
          <button class="quick-card q-mint" @click="navigate('lookup')"><span class="quick-icon"><Search/></span><h3>查詢進度</h3><p>輸入單號，了解處理狀態</p><span class="quick-arrow"><ArrowRight/></span></button>
          <button class="quick-card q-lemon" @click="navigate('booking')"><span class="quick-icon"><CalendarDays/></span><h3>線上預約</h3><p>先選日期，安排到店時間</p><span class="quick-arrow"><ArrowRight/></span></button>
          <button class="quick-card q-lavender" @click="navigate('estimate')"><span class="quick-icon"><Calculator/></span><h3>費用評估</h3><p>試選項目，查看模擬金額</p><span class="quick-arrow"><ArrowRight/></span></button>
        </div></section>
        <section class="service-section"><div class="container"><div class="section-heading inline-heading"><div><span class="section-kicker">OUR SERVICES</span><h2>你的設備，我們都在乎</h2><p>先選擇設備類型，再了解適合的服務。</p></div><button class="text-link" @click="navigate('services')">查看所有服務 <ArrowRight :size="20"/></button></div><div class="service-grid"><button v-for="service in serviceCatalog.slice(0,4)" :key="service.id" class="service-card" @click="navigate('services')"><span class="service-illustration" :class="service.tone"><component :is="service.icon" :size="44" :stroke-width="1.65"/></span><h3>{{service.name}}</h3><p>{{service.brief}}</p><span class="service-link">了解服務 <ArrowRight :size="18"/></span></button></div></div></section>
        <section class="flow-section container"><div class="section-heading center"><span class="section-kicker">HOW IT WORKS</span><h2>送修其實很簡單</h2><p>只要三個步驟，就能開始處理設備問題。</p></div><div class="steps"><div class="step"><span class="step-num">01</span><span class="step-icon peach"><ClipboardList/></span><h3>填寫需求</h3><p>選擇設備並描述遇到的問題</p></div><div class="steps-arrow">→</div><div class="step"><span class="step-num">02</span><span class="step-icon mint"><Phone/></span><h3>門市聯繫</h3><p>由人員確認問題與送修安排</p></div><div class="steps-arrow">→</div><div class="step"><span class="step-num">03</span><span class="step-icon lemon"><CheckCircle2/></span><h3>檢測維修</h3><p>依實際檢測情況溝通報價</p></div></div></section>
        <section class="cta-wrap"><div class="container"><div class="cta-panel"><div><span class="section-kicker">READY WHEN YOU ARE</span><h2>科技遇到難題，我們一起解決。</h2><p>不確定哪裡壞了也沒關係，先告訴我們你的狀況。</p></div><button class="btn btn-dark" @click="navigate('register')">開始送修登記 <ArrowRight :size="20"/></button></div></div></section>
      </template>

      <template v-if="page==='services'"><section class="page-hero container"><span class="section-kicker">SERVICE CENTER</span><h1>每一種設備，都值得細心照顧。</h1><p>無論是設備故障或例行保養，都可以從這裡找到服務方向。</p></section><section class="container page-content"><div class="filter-row"><button v-for="f in ['全部','筆電','手機','桌機','平板','其他']" :key="f" class="filter-pill" :class="{selected:serviceFilter===f}" @click="serviceFilter=f">{{f}}</button></div><div class="service-grid all-services"><article v-for="s in filteredServices" :key="s.id" class="service-card"><span class="service-illustration" :class="s.tone"><component :is="s.icon" :size="44" :stroke-width="1.65"/></span><h3>{{s.name}}</h3><p>{{s.brief}}</p><button class="service-link" @click="form.device = s.tag==='其他'?'筆電':s.tag; navigate('register')">填寫需求 <ArrowRight :size="18"/></button></article></div><div class="notice"><Info :size="22"/><span>實際維修項目、料件與報價須經門市確認。此展示站不代表已受理正式維修服務。</span></div></section></template>

      <template v-if="page==='register'"><section class="page-hero container compact"><span class="section-kicker">REPAIR REQUEST</span><h1>線上送修登記</h1><p>簡單填寫設備問題，讓需求更容易被理解。</p></section><section class="container form-layout"><div class="form-main"><div class="progress-steps"><span :class="{current:submitStep>=1}">1　設備問題</span><span :class="{current:submitStep>=2}">2　聯絡資料</span><span :class="{current:submitStep>=3}">3　建立完成</span></div><div class="form-panel" v-if="submitStep===1"><span class="form-step">STEP 01</span><h2>先聊聊設備怎麼了</h2><label class="field"><span>設備種類</span><select v-model="form.device"><option>筆電</option><option>手機</option><option>桌機</option><option>平板</option><option>其他</option></select></label><label class="field"><span>遇到什麼問題？ <b>*</b></span><textarea v-model="form.issue" rows="5" placeholder="例如：筆電開機後畫面全黑、風扇聲音很大……"></textarea></label><label class="field"><span>補充說明（選填）</span><textarea v-model="form.note" rows="3" placeholder="設備品牌、型號、使用狀況等"></textarea></label><div class="form-actions"><button class="btn btn-primary" @click="form.issue.trim()?submitStep=2:notify('請先描述設備遇到的問題')">下一步 <ArrowRight :size="19"/></button></div></div>
      <div class="form-panel" v-if="submitStep===2"><span class="form-step">STEP 02</span><h2>我們怎麼聯絡你？</h2><label class="field"><span>聯絡姓名 <b>*</b></span><input v-model="form.name" placeholder="請輸入姓名" autocomplete="name" /></label><label class="field"><span>手機號碼 <b>*</b></span><input v-model="form.phone" type="tel" inputmode="tel" maxlength="10" placeholder="09xxxxxxxx" autocomplete="tel" /></label><label class="field"><span>偏好聯繫方式</span><select v-model="form.contact"><option>電話</option><option>簡訊</option></select></label><div class="agreement"><h3><ShieldCheck :size="21"/> 送修注意事項</h3><p>送修前請先備份重要資料。實際拆機、風險說明、報價同意、維修保固與取件條件，應以門市正式條款及雙方確認內容為準。</p><a href="tel:0423924505">致電確認永茂3C維修條款 <Phone :size="15"/></a><label class="check-label"><input type="checkbox" v-model="accepted"/> 我已閱讀上述提醒，了解這是展示版登記，不會直接送到門市。</label></div><div class="form-actions between"><button class="btn btn-outline" @click="submitStep=1"><ArrowLeft :size="18"/> 上一步</button><button class="btn btn-primary" @click="newTicket">建立展示登記 <ArrowRight :size="19"/></button></div></div>
      <div class="form-panel success-panel" v-if="submitStep===3"><span class="success-circle"><CheckCircle2 :size="48"/></span><h2>展示登記建立成功！</h2><p>以下資料只存放在你的瀏覽器，<strong>尚未送達 永茂3C門市</strong>。</p><div class="ticket-number">{{ inquiryId }}</div><div class="form-actions center-actions"><button class="btn btn-primary" @click="navigate('lookup');lookup()">查看展示進度 <ArrowRight :size="19"/></button><button class="btn btn-outline" @click="resetTicket">再建立一筆</button></div><a class="original-link" href="tel:0423924505">致電永茂3C確認送修 <Phone :size="16"/></a></div>
      </div><aside class="side-panel"><div class="side-badge"><HeartHandshake :size="26"/></div><h3>放心，我們一步一步來</h3><p>不必懂電腦術語，只要描述你看到的狀況即可。</p><div class="side-list"><span><CheckCircle2 :size="18"/> 資料分步填寫</span><span><CheckCircle2 :size="18"/> 可回上一步修改</span><span><CheckCircle2 :size="18"/> 不收集開機密碼</span></div><div class="demo-warning"><Info :size="18"/> 此版本僅為前端展示，並非門市受理系統。</div></aside></section></template>

      <template v-if="page==='lookup'"><section class="page-hero container compact"><span class="section-kicker">REPAIR TRACKING</span><h1>維修進度查詢</h1><p>輸入展示單號與手機號碼，即可查看本機建立的登記。</p></section><section class="container narrow-page"><div class="form-panel"><div class="demo-notice"><Info :size="19"/> 目前僅支援查詢本瀏覽器建立的展示單號，尚未串接門市正式維修資料。</div><label class="field"><span>展示維修單號</span><input v-model="inquiryId" placeholder="DEMO-xxxxxxxx" /></label><label class="field"><span>登記手機號碼</span><input v-model="inquiryPhone" type="tel" maxlength="10" placeholder="09xxxxxxxx" /></label><button class="btn btn-primary wide" @click="lookup"><Search :size="20"/> 查詢進度</button><div v-if="inquiryResult===false" class="lookup-error">查無此展示資料，請確認單號、電話與是否使用相同瀏覽器。</div><div v-else-if="inquiryResult" class="lookup-result"><span class="lookup-status"><CheckCircle2 :size="18"/> {{ inquiryResult.status }}</span><h3>{{ inquiryResult.device }} · {{ inquiryResult.issue }}</h3><p>建立日期：{{ inquiryResult.date }}</p><p>下一步：正式版需由門市接件、檢測並更新狀態；展示資料不會自動變更。</p></div><p class="form-hint">查詢正式進度請使用 <a href="tel:0423924505">永茂3C門市 <Phone :size="14"/></a>。</p></div></section></template>

      <template v-if="page==='booking'"><section class="page-hero container compact"><span class="section-kicker">BOOK A VISIT</span><h1>安排一個方便的時間</h1><p>先填好到店需求，正式預約仍須由門市確認。</p></section><section class="container narrow-page"><form class="form-panel" @submit.prevent="submitBooking"><div class="demo-notice"><Info :size="19"/> 展示預約僅儲存在目前瀏覽器，沒有保留門市時段。</div><div class="two-col"><label class="field"><span>姓名 *</span><input v-model="book.name" placeholder="您的姓名" required /></label><label class="field"><span>手機號碼 *</span><input v-model="book.phone" type="tel" maxlength="10" placeholder="09xxxxxxxx" required /></label><label class="field"><span>希望日期 *</span><input v-model="book.date" type="date" :min="new Date().toLocaleDateString('en-CA')" required /></label><label class="field"><span>希望時間 *</span><select v-model="book.time"><option v-for="time in ['11:00','12:00','13:00','14:00','15:00','16:00','17:00','18:00','19:00','20:00']" :key="time">{{time}}</option></select></label></div><label class="field"><span>設備種類</span><select v-model="book.device"><option>筆電</option><option>手機</option><option>桌機</option><option>平板</option><option>其他</option></select></label><label class="field"><span>需求說明</span><textarea v-model="book.note" rows="3" placeholder="簡單描述要處理的問題"></textarea></label><button class="btn btn-primary wide" type="submit"><CalendarDays :size="20"/> 儲存展示預約</button><p class="form-hint">正式預約請使用 <a href="tel:0423924505">電話洽詢永茂3C <Phone :size="14"/></a>。</p><div v-if="reservations.length" class="saved-bookings"><h3>本機展示預約紀錄</h3><div v-for="r in reservations.slice(0,3)" :key="r.id" class="booking-line"><b>{{r.date}}　{{r.time}}</b><span>{{r.device}} · {{r.id}}</span></div></div></form></section></template>

      <template v-if="page==='estimate'">
        <section class="page-hero container compact"><span class="section-kicker">REPAIR COST SIMULATOR</span><h1>維修費用試算</h1><p>選擇設備及多個維修項目，快速了解模擬預估費用。</p></section>
        <section class="container narrow-page"><div class="form-panel">
          <div class="demo-notice"><Info :size="20"/><span><strong>模擬價目表，非正式報價。</strong> 以下金額僅供網站功能展示，不代表永茂3C已核定的收費，請勿視為維修承諾。</span></div>
          <label class="field"><span>01　選擇設備類型</span><select v-model="estimatedDevice"><option v-for="dev in Object.keys(priceCatalog)" :key="dev" :value="dev">{{dev}}</option></select></label>
          <div class="estimate-list-head"><h2>02　選擇維修項目</h2><span>可複選，已選 {{selectedItems.length}} 項</span></div>
          <div class="price-options"><label v-for="item in availablePrices" :key="item.name" class="price-option" :class="{'price-option-active':selectedItems.includes(item.name)}">
            <input type="checkbox" :checked="selectedItems.includes(item.name)" @change="togglePrice(item.name)"/>
            <span class="price-option-name">{{item.name}}<small v-if="item.labor">僅工資，不含零件</small></span>
            <strong>{{nt(item.price)}}<small v-if="item.starting"> 起</small></strong>
          </label></div>
          <div class="estimate-total-box" aria-live="polite"><div><span>模擬{{estimateIsMinimum?'起始':''}}合計</span><small>已選 {{selectedItems.length}} 項</small></div><strong>{{nt(estimateTotal)}}{{estimateIsMinimum?' 起':''}}</strong></div>
          <p class="estimate-fine-print">各項維修價格是示意金額，部分工項可能重疊、不一定逐項累加；零件、品牌型號、故障原因與檢測費折抵方式均須門市確認。未提供正式的檢測費折抵或折扣承諾。</p>
          <div class="estimate-actions"><button class="btn btn-outline" :disabled="!selectedItems.length" @click="selectedItems=[]">清除選項</button><button class="btn btn-primary" :disabled="!selectedItems.length" @click="copyEstimate"><ClipboardList :size="19"/> 複製試算結果</button></div>
          <div class="estimate-next"><h3>需要進一步確認維修狀況？</h3><p>可以先填寫展示送修表單，或直接致電門市詢問正式報價。</p><div class="estimate-actions"><button class="btn btn-outline" @click="form.device = estimatedDevice==='清潔／軟體'?'筆電':estimatedDevice; navigate('register')">前往送修登記 <ArrowRight :size="19"/></button><a class="btn btn-primary" href="tel:0423924505"><Phone :size="19"/> (04) 2392-4505</a></div></div>
        </div></section>
      </template>

      <template v-if="page==='faq'"><section class="page-hero container compact"><span class="section-kicker">HELP CENTER</span><h1>你可能也想知道</h1><p>關於登記、預約與維修流程，這裡先回答常見疑問。</p></section><section class="container narrow-page"><div class="faq-list"><div v-for="([q,a],i) in questions" :key="q" class="faq-item"><button :aria-expanded="activeFaq===i" @click="activeFaq=activeFaq===i?-1:i"><span>{{q}}</span><Minus v-if="activeFaq===i" :size="22"/><Plus v-else :size="22"/></button><p v-if="activeFaq===i">{{a}}</p></div></div><div class="notice"><Info :size="22"/><span>更多真實維修知識可參考 <a href="tel:0423924505">永茂3C門市</a>。</span></div></section></template>
    </main>

    <footer class="footer"><div class="container footer-grid"><div><div class="footer-brand"><span class="brand-icon"><Wrench :size="23"/></span><b>永茂3C</b></div><p>有溫度的科技服務，讓每次維修都更安心。</p><span class="footer-demo">Vue 3 前端展示版 · 尚未串接正式受理系統</span></div><div><h4>快速服務</h4><button @click="navigate('register')">線上送修</button><button @click="navigate('lookup')">進度查詢</button><button @click="navigate('booking')">線上預約</button></div><div><h4>網站資訊</h4><button @click="navigate('services')">維修服務</button><button @click="navigate('faq')">常見問題</button><a href="tel:0423924505">聯絡門市 <Phone :size="14"/></a></div><div><h4>太平坪林門市</h4><p><MapPin :size="17"/> 臺中市太平區坪林里中山路二段 57號</p><a href="tel:0423924505"><Phone :size="17"/> (04) 2392-4505</a><a href="https://www.google.com/maps/search/?api=1&amp;query=%E8%87%BA%E4%B8%AD%E5%B8%82%E5%A4%AA%E5%B9%B3%E5%8D%80%E5%9D%AA%E6%9E%97%E9%87%8C%E4%B8%AD%E5%B1%B1%E8%B7%AF%E4%BA%8C%E6%AE%B557%E8%99%9F" target="_blank" rel="noopener"><MapPin :size="16"/> 地圖位置</a></div></div><div class="container footer-bottom">© 永茂3C · Vue 3 前端展示版，正式服務請致電門市確認。</div></footer>
    <div v-if="toast" role="status" class="toast"><Info :size="18"/>{{toast}}</div>
  </div>
</template>
