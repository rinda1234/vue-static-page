<script setup lang="ts">
/**
 * App.vue — Chuseok (한가위) interactive landing
 * Vue 3 + TypeScript + Vite
 * - 달 배경/등불/소원/음성/레시피/송편/강강술래/스토리/갤러리/레시피카드
 * - ✅ 제기차기(간단 물리 시뮬레이션) 추가
 */

import { ref, reactive, onMounted, onBeforeUnmount, nextTick } from 'vue'

/* --------------------------
   Asset URLs (Vite friendly)
   Put your images in src/assets/han-gawi/
   -------------------------- */
const IMG = {
  moon: new URL('./assets/han-gawi/moon.jpg', import.meta.url).href,
  songpyeon: new URL('./assets/han-gawi/songpyeon.jpg', import.meta.url).href,
  jeon: new URL('./assets/han-gawi/jeon.jpg', import.meta.url).href,
  hangwa: new URL('./assets/han-gawi/hangwa.jpg', import.meta.url).href,
  yut: new URL('./assets/han-gawi/yut.jpg', import.meta.url).href,
  ganggang: new URL('./assets/han-gawi/ganggang.jpg', import.meta.url).href,
  jegi: new URL('./assets/han-gawi/jegichagi.jpg', import.meta.url).href,
  gallery1: new URL('./assets/han-gawi/gallery1.jpg', import.meta.url).href,
  gallery2: new URL('./assets/han-gawi/gallery2.jpg', import.meta.url).href,
}
const BGM = new URL('./assets/han-gawi/chuseok_bgm.mp3', import.meta.url).href

/* --------------------------
   Reactive state
   -------------------------- */
const title = ref('달빛 아래, 우리 마음의 한가위')
const subtitle = ref('전통의 맛과 놀이, 기억을 웹에서 체험하세요.')

const wish = ref('')
const wishes = ref<string[]>([])
const submittedWish = ref('')

const foods = reactive([
  {
    id: 'songpyeon',
    name: '송편',
    desc: '한가위를 대표하는 반달 모양 떡으로, 솔잎 향이 은은하게 배어드는 풍요의 상징입니다.',
    img: IMG.songpyeon,
    recipe:
        `① 쌀가루에 따뜻한 물을 부어 익반죽합니다.

② 반죽을 작게 떼어 손바닥에 올리고, 팥·깨·콩 등 속재료를 넣습니다.

③ 반달 모양으로 빚어 모양을 다듬습니다.

④ 솔잎을 깔고 송편을 올린 뒤 김이 오른 찜기에 찝니다.

⑤ 솔잎 향이 배면 꺼내 식혀 즐깁니다.`
  },

  {
    id: 'jeon',
    name: '전',
    desc: '재료를 밀가루와 달걀옷에 입혀 지져낸 음식으로, 명절에 가족이 함께 부치며 정을 나누는 전통의 맛입니다.',
    img: IMG.jeon,
    recipe:
        `① 재료(호박, 동태, 깻잎 등)를 먹기 좋은 크기로 썹니다.

② 밀가루를 묻히고 달걀을 풀어 옷을 입힙니다.

③ 팬에 기름을 두르고 중불에서 앞뒤로 노릇하게 부칩니다.

④ 키친타월에 올려 기름기를 뺍니다.

⑤ 따뜻할 때 간장 양념과 함께 즐깁니다.`
  },

  {
    id: 'hangwa',
    name: '한과',
    desc: '쌀가루와 꿀, 기름으로 만든 전통 과자로, 바삭하고 달콤한 맛에 조상의 지혜가 담겨 있습니다.',
    img: IMG.hangwa,
    recipe:
        `① 찹쌀가루나 멥쌀가루를 반죽해 적당한 크기로 빚습니다.

② 낮은 온도의 기름에 넣어 바삭하게 튀깁니다.

③ 꿀이나 조청을 약하게 끓여 시럽을 만듭니다.

④ 튀긴 과자를 시럽에 버무린 뒤 고명(깨, 잣 등)을 올립니다.

⑤ 식혀서 바삭하게 굳히면 완성됩니다.`
  },
])


/* --------------------------
   modal / ui
   -------------------------- */
const recipeModal = ref<{ open: boolean; foodId?: string }>({ open: false })
function openRecipe(id: string) { recipeModal.value = { open: true, foodId: id } }
function closeRecipe() { recipeModal.value.open = false }

/* --------------------------
   Speech & BGM
   -------------------------- */
let bgAudio: HTMLAudioElement | null = null
const bgPlaying = ref(false)

function initBgm() {
  if (!bgAudio) {
    bgAudio = new Audio(BGM)
    bgAudio.loop = true
    bgAudio.volume = 0.35
  }
}
function toggleBgm() {
  initBgm()
  if (!bgAudio) return
  if (bgPlaying.value) { bgAudio.pause(); bgPlaying.value = false }
  else {
    const p = bgAudio.play()
    p?.catch(() => {})
    bgPlaying.value = true
  }
}
function speak(text: string) {
  if (!('speechSynthesis' in window)) return
  const u = new SpeechSynthesisUtterance(text)
  u.lang = 'ko-KR'
  u.rate = 1.0
  window.speechSynthesis.cancel()
  window.speechSynthesis.speak(u)
}

/* --------------------------
   Wish handling (localStorage)
   -------------------------- */
const WISH_KEY = 'chuseok:wishes'
function loadWishes() {
  try {
    const raw = localStorage.getItem(WISH_KEY)
    if (raw) wishes.value = JSON.parse(raw)
  } catch (e) { console.warn(e) }
}
function saveWishes() { localStorage.setItem(WISH_KEY, JSON.stringify(wishes.value)) }
function submitWish() {
  if (!wish.value.trim()) return

  // 새 소원 추가
  wishes.value.unshift(wish.value.trim())
  submittedWish.value = wish.value.trim()

  // ✅ 최근 5개까지만 유지
  if (wishes.value.length > 5) {
    wishes.value = wishes.value.slice(0, 5)
  }

  saveWishes()
  speak(`${submittedWish.value} 라는 소원이 달빛에 띄워졌습니다.`)
  wish.value = ''
}

/* --------------------------
   Lanterns Canvas (robust resize)
   -------------------------- */
// let lanternCanvasEl: HTMLCanvasElement | null = null
let lanternRAF = 0
const lanterns: Array<any> = []
function setupLanterns(canvas: HTMLCanvasElement) {
  // lanternCanvasEl = canvas
  const ctx = (canvas as HTMLCanvasElement).getContext('2d')!

  function resize() {
    const DPR = devicePixelRatio || 1
    canvas.width = Math.floor(window.innerWidth * DPR)
    canvas.height = Math.floor(300 * DPR)
    canvas.style.width = '100%'
    canvas.style.height = '300px'
    ctx.setTransform(DPR, 0, 0, DPR, 0, 0)
  }
  function init() {
    resize()
    lanterns.length = 0
    for (let i = 0; i < 14; i++) {
      lanterns.push({
        x: Math.random() * window.innerWidth,
        y: 300 + Math.random() * 100,
        vx: (Math.random() - 0.5) * 0.4,
        vy: - (0.2 + Math.random() * 0.6),
        r: 6 + Math.random() * 16,
        alpha: 0.4 + Math.random() * 0.6
      })
    }
  }
  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    ctx.save()
    ctx.globalCompositeOperation = 'lighter'
    for (const l of lanterns) {
      ctx.beginPath()
      const grad = ctx.createRadialGradient(l.x, l.y, 0, l.x, l.y, l.r)
      grad.addColorStop(0, `rgba(255,230,180,${l.alpha})`)
      grad.addColorStop(1, `rgba(255,140,80,0)`)
      ctx.fillStyle = grad
      ctx.arc(l.x, l.y, l.r, 0, Math.PI * 2)
      ctx.fill()
      l.x += l.vx
      l.y += l.vy
      l.vx += (Math.random() - 0.5) * 0.02
      l.vy -= 0.001
      if (l.y < -60) { l.y = 320; l.x = Math.random() * window.innerWidth }
    }
    ctx.restore()
    lanternRAF = requestAnimationFrame(draw)
  }
  init()
  window.addEventListener('resize', resize)
  draw()
  return () => {
    cancelAnimationFrame(lanternRAF)
    window.removeEventListener('resize', resize)
  }
}

/* --------------------------
   Moon Canvas (parallax + resize)
   -------------------------- */
// let moonCanvasEl: HTMLCanvasElement | null = null
let moonRAF = 0
function setupMoon(canvas: HTMLCanvasElement) {
  // moonCanvasEl = canvas
  const ctx = canvas.getContext('2d')!
  function resize() {
    const DPR = devicePixelRatio || 1
    canvas.width = Math.floor(window.innerWidth * DPR)
    canvas.height = Math.floor(window.innerHeight * DPR)
    canvas.style.width = '100%'
    canvas.style.height = '100vh'
    ctx.setTransform(DPR, 0, 0, DPR, 0, 0)
  }
  let t = 0
  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    const w = canvas.width / (devicePixelRatio || 1)
    const h = canvas.height / (devicePixelRatio || 1)
    // gentle radial glow
    const grd = ctx.createRadialGradient(w/2, h/3, 50, w/2, h/3, Math.max(w,h))
    grd.addColorStop(0, 'rgba(255,247,210,0.55)')
    grd.addColorStop(0.5, 'rgba(255,245,200,0.12)')
    grd.addColorStop(1, 'rgba(10,12,30,0.9)')
    ctx.fillStyle = grd
    ctx.fillRect(0, 0, w, h)
    // moon disc (soft)
    const mx = w/2 + Math.sin(t/200) * 30
    const my = h/3 + Math.cos(t/300) * 10
    const r = Math.min(160, w*0.12)
    const g2 = ctx.createRadialGradient(mx, my, r*0.1, mx, my, r*1.6)
    g2.addColorStop(0, 'rgba(255,250,220,1)')
    g2.addColorStop(1, 'rgba(255,245,200,0)')
    ctx.beginPath()
    ctx.fillStyle = g2
    ctx.arc(mx, my, r, 0, Math.PI*2)
    ctx.fill()
    t += 1
    moonRAF = requestAnimationFrame(draw)
  }
  resize()
  window.addEventListener('resize', resize)
  draw()
  return () => {
    cancelAnimationFrame(moonRAF)
    window.removeEventListener('resize', resize)
  }
}

/* --------------------------
   Simple Yut roll (윷 놀이)
   -------------------------- */
const yutResult = ref<string | null>(null)
function rollYut() {
  const outcomes = ['도','개','걸','윷','모']
  const idx = Math.floor(Math.random() * outcomes.length)
  yutResult.value = outcomes[idx] ?? null
  speak(`결과는 ${yutResult.value} 입니다`)
}

/* --------------------------
   Simple Ganggang visualize (circle dancers)
   -------------------------- */
const ganggangSpin = ref(false)
function toggleGanggang() { ganggangSpin.value = !ganggangSpin.value }

/* --------------------------
   Story upload -> localStorage (images as dataURL)
   -------------------------- */
const stories = ref<Array<{ id: string; text: string; img?: string; date: string }>>([])
const STORY_KEY = 'chuseok:stories'
function loadStories() {
  try {
    const raw = localStorage.getItem(STORY_KEY)
    if (raw) stories.value = JSON.parse(raw)
  } catch (e) { console.warn(e) }
}
function saveStories() { localStorage.setItem(STORY_KEY, JSON.stringify(stories.value)) }
async function handleStoryUpload(e: Event) {
  const input = e.target as HTMLInputElement
  if (!input.files || !input.files[0]) return
  const file = input.files[0]
  const reader = new FileReader()
  reader.onload = () => {
    const data = reader.result as string
    stories.value.unshift({ id: Date.now().toString(), text: storyText.value || '한가위 추억', img: data, date: new Date().toISOString() })
    storyText.value = ''
    saveStories()
  }
  reader.readAsDataURL(file)
}
const storyText = ref('')

/* --------------------------
   Color extraction from first story image (dominant-ish)
   -------------------------- */
const dominantColor = ref<string>('#f7c46b')
async function updateDominantFromLatestStory() {
  const s = stories.value[0]
  if (!s?.img) return
  const img = new Image()
  img.crossOrigin = 'anonymous'
  img.src = s.img
  await new Promise((res) => (img.onload = res))
  const c = document.createElement('canvas')
  const ctx = c.getContext('2d')!
  c.width = Math.min(100, img.width)
  c.height = Math.min(100, img.height)
  ctx.drawImage(img, 0, 0, c.width, c.height)
  const data = ctx.getImageData(0,0,c.width,c.height).data
  let r=0,g=0,b=0,count=0
  for (let i=0;i<data.length;i+=4){ r+=data[i]; g+=data[i+1]; b+=data[i+2]; count++ }
  r = Math.round(r/count); g = Math.round(g/count); b = Math.round(b/count)
  dominantColor.value = `rgb(${r},${g},${b})`
}

/* --------------------------
   Recipe card -> canvas PNG download
   -------------------------- */
async function downloadRecipeCard(foodId: string) {
  const f = foods.find(x => x.id === foodId)
  if (!f) return

  const c = document.createElement('canvas')
  c.width = 800
  c.height = 480
  const ctx = c.getContext('2d')!

  ctx.fillStyle = '#fffef8'
  ctx.fillRect(0, 0, c.width, c.height)

  ctx.fillStyle = '#2b2b2b'
  ctx.font = '28px serif'
  ctx.fillText(`${f.name} 레시피`, 40, 60)

  // ✅ 줄바꿈 적용 (핵심 부분)
  ctx.font = '18px serif'
  const lines = f.recipe.split('\n')   // 🔹 줄마다 분리
  let y = 110
  for (const line of lines) {
    ctx.fillText(line.trim(), 40, y)
    y += 28  // 줄 간격
  }

  // 이미지
  const img = new Image()
  img.crossOrigin = 'anonymous'
  img.src = f.img
  await new Promise((res) => (img.onload = res))
  ctx.drawImage(img, c.width - 260, 60, 220, 140)

  // 다운로드
  const url = c.toDataURL('image/png')
  const a = document.createElement('a')
  a.href = url
  a.download = `${f.id}_recipe.png`
  a.click()
}

/* --------------------------
   송편(간단 그물망 물리) — 기존 로직 유지
   -------------------------- */
onMounted(() => {
  loadWishes()
  loadStories()
  // setup canvases after nextTick so refs exist
  nextTick(() => {
    const lc = document.getElementById('lantern-canvas') as HTMLCanvasElement | null
    if (lc) cleanupLanterns = setupLanterns(lc)
    const mc = document.getElementById('moon-canvas') as HTMLCanvasElement | null
    if (mc) cleanupMoon = setupMoon(mc)

    // 송편
    const canvas = document.getElementById('songpyeon-canvas') as HTMLCanvasElement | null
    if (!canvas) return
    if (canvas) {
      const ctx = canvas.getContext('2d')!
      const points:any[] = []
      const cx = canvas.width/2, cy = canvas.height/2
      const radius = 60
      const detail = 1028
      for (let i=0;i<detail;i++){
        const angle = (i/detail)*Math.PI*2
        points.push({
          x: cx + Math.cos(angle)*radius,
          y: cy + Math.sin(angle)*radius*0.7,
          ox: cx + Math.cos(angle)*radius,
          oy: cy + Math.sin(angle)*radius*0.7,
          vx: 0, vy: 0
        })
      }
      let dragging = false
      canvas.addEventListener('mousedown', ()=>dragging=true)
      window.addEventListener('mouseup', ()=>dragging=false)
      canvas.addEventListener('mousemove', (e)=>{
        if (!dragging) return
        const rect = canvas.getBoundingClientRect()
        const mx = e.clientX - rect.left
        const my = e.clientY - rect.top
        for (const p of points){
          const dx = p.x - mx, dy = p.y - my
          const dist = Math.sqrt(dx*dx + dy*dy)
          if (dist < 50){ p.vx += (mx - p.x)*0.015; p.vy += (my - p.y)*0.015 }
        }
      })
      function animate(){
        ctx.clearRect(0,0,canvas.width,canvas.height)
        for (const p of points){
          const dx = p.ox - p.x, dy = p.oy - p.y
          p.vx += dx*0.26; p.vy += dy*0.26
          p.vx *= 0.90; p.vy *= 0.90
          p.x += p.vx;  p.y += p.vy
        }
        // 외곽을 부드럽게
        ctx.beginPath()
        for (let i=0;i<points.length;i++){
          const a=points[i], b=points[(i+1)%points.length]
          const mx=(a.x+b.x)/2, my=(a.y+b.y)/2
          if(i===0) ctx.moveTo(mx,my); else ctx.quadraticCurveTo(a.x,a.y,mx,my)
        }
        ctx.closePath()
        const grd = ctx.createLinearGradient(0,cy-50,0,cy+60)
        grd.addColorStop(0,'#b8f3c5'); grd.addColorStop(1,'#5ca782')
        ctx.fillStyle = grd; ctx.fill()
        requestAnimationFrame(animate)
      }
      animate()
    }

    // ✅ 제기차기 캔버스 초기화
    const jc = document.getElementById('jegi-canvas') as HTMLCanvasElement | null
    if (jc) initJegi(jc)
  })
})

onBeforeUnmount(() => {
  if (cleanupLanterns) cleanupLanterns()
  if (cleanupMoon) cleanupMoon()
})

/* --------------------------
   Parallax: simple scroll listener
   -------------------------- */
function onScrollParallax() {
  const v = window.scrollY
  const hero = document.querySelector('.hero-foreground') as HTMLElement | null
  if (hero) hero.style.transform = `translateY(${v * 0.08}px)`
}
window.addEventListener('scroll', onScrollParallax)

/* --------------------------
   ✅ 제기차기: 간단 물리 시뮬레이션
   - 버튼/클릭/터치로 튕김
   - 중력/반발/회전(간단) 구현
   -------------------------- */
const jegiScore = ref(0)
let jegiKickPower = 10 // 차기 힘 (버튼/클릭 공통)
function setJegiPower(p:number){ jegiKickPower = Math.max(6, Math.min(16, p)) }

function initJegi(canvas: HTMLCanvasElement){
  const ctx = canvas.getContext('2d')!
  const W = canvas.width, H = canvas.height
  const groundY = H - 28

  const jegi = {
    x: W/2,
    y: groundY,
    vx: 0,
    vy: 0,
    r: 16,
    ay: 0.42,    // 중력
    bounce: 0.35,// 바닥 반발
    spin: 0,     // 회전
    angle: 0,    // 회전각
    flying: false
  }

  function draw() {
    // 배경/땅
    ctx.clearRect(0,0,W,H)
    const bg = ctx.createRadialGradient(W/2,H/2,10,W/2,H/2,220)
    bg.addColorStop(0,'#0b0e18'); bg.addColorStop(1,'#02030a')
    ctx.fillStyle = bg; ctx.fillRect(0,0,W,H)
    ctx.fillStyle = '#182030'; ctx.fillRect(0,groundY,W,H-groundY)

    // 제기 본체
    ctx.save()
    ctx.translate(jegi.x, jegi.y)
    ctx.rotate(jegi.angle)
    ctx.beginPath()
    ctx.arc(0, 0, jegi.r, 0, Math.PI*2)
    ctx.fillStyle = '#f4c542'
    ctx.fill()
    // 깃털
    ctx.beginPath()
    ctx.moveTo(0, -jegi.r)
    ctx.lineTo(-10, -jegi.r-20)
    ctx.lineTo(10, -jegi.r-20)
    ctx.closePath()
    ctx.fillStyle = '#ffffff'
    ctx.fill()
    ctx.restore()

    // 그림자
    const shadowScale = 1 - Math.min(1, (jegi.y - (groundY-80)) / 80)
    ctx.save()
    ctx.globalAlpha = 0.25 * shadowScale
    ctx.beginPath()
    ctx.ellipse(jegi.x, groundY+6, 22*shadowScale, 8*shadowScale, 0, 0, Math.PI*2)
    ctx.fillStyle = '#000'; ctx.fill()
    ctx.restore()
  }

  function step() {
    if (jegi.flying) {
      jegi.vy += jegi.ay
      jegi.y += jegi.vy
      jegi.x += jegi.vx
      jegi.angle += jegi.spin

      // 좌우 벽 반발
      if (jegi.x < jegi.r) { jegi.x = jegi.r; jegi.vx *= -0.25 }
      if (jegi.x > W - jegi.r) { jegi.x = W - jegi.r; jegi.vx *= -0.25 }

      // 바닥 닿음 처리 (단, 점수는 유지)
      if (jegi.y >= groundY) {
        jegi.y = groundY
        jegi.vy = 0
        jegi.vx = 0
        jegi.spin = 0
        jegi.angle = 0
        jegi.flying = false
      }
    }

    draw()
    requestAnimationFrame(step)
  }

  function kick(power = jegiKickPower) {
    // 제기가 멈춘 상태에서만 점수 초기화
    if (!jegi.flying && jegi.y >= groundY) {
      jegiScore.value = 0   // ✅ 새 라운드 시작
    }

    // 찰 때마다 상승
    jegi.vy = -power
    jegi.vx = (Math.random() - 0.5) * 2.0
    jegi.spin = (Math.random() - 0.5) * 0.08
    jegi.flying = true
    jegiScore.value += 1
  }


  // 이벤트(버튼은 외부 메서드 호출, 캔버스도 클릭/터치로 차기)
  canvas.addEventListener('click', () => kick())
  canvas.addEventListener('touchstart', (e) => { e.preventDefault(); kick() }, { passive: false })

  // 외부에서 쓸 수 있도록 바인딩
  ;(kickJegi as any)._impl = kick as () => void

  draw()
  step()
}

// 외부 버튼용 래퍼
function kickJegi() {
  const fn = (kickJegi as any)._impl as (() => void) | undefined
  if (fn) fn()
}
/* --------------------------
   Lifecycle cleanup refs
   -------------------------- */
let cleanupLanterns: (() => void) | null = null
let cleanupMoon: (() => void) | null = null
</script>

<template>
  <div class="app-root" :style="{ '--accent': dominantColor } as any">
    <!-- MOON canvas -->
    <canvas id="moon-canvas" class="moon-canvas"></canvas>

    <!-- HERO -->
    <header class="hero">
      <div class="hero-inner">
        <h1 class="title">{{ title }}</h1>
        <p class="subtitle">{{ subtitle }}</p>

        <div class="hero-controls">
          <input v-model="wish" @keyup.enter="submitWish" class="wish-input" placeholder="소원 입력 후 엔터" />
          <button @click="submitWish" class="btn">달에게 띄우기</button>
          <button @click="toggleBgm" class="btn muted">{{ bgPlaying ? '음악 끄기' : '음악 켜기' }}</button>
          <button @click="speak(title)" class="btn">읽어줘</button>
        </div>
        <!-- 달에 떠 있는 소원들 -->
        <div class="wishes-on-moon">
          <div
              v-for="(w, i) in wishes"
              :key="i"
              class="floating-wish"
              :style="{
                top: `${20 + i * 40}px`,
                left: `${50 + (i % 2) * 80}px`,
                opacity: 1 - i * 0.15
              }"
          >
            🌙 {{ w }}
          </div>
        </div>
        <p v-if="submittedWish" class="wish-announce">“{{ submittedWish }}” — 달빛에 띄워졌습니다.</p>

        <!-- small stats / actions -->
        <div class="hero-actions">
          <button @click="openRecipe('songpyeon')" class="small-btn">송편 레시피 보기</button>
          <button @click="openRecipe('jeon')" class="small-btn">전 레시피 보기</button>
          <button @click="openRecipe('hangwa')" class="small-btn">한과 레시피 보기</button>
        </div>
      </div>

      <!-- lanterns canvas on bottom of hero -->
      <canvas id="lantern-canvas" class="lantern-canvas" aria-hidden="true"></canvas>
    </header>

    <!-- SECTIONS -->
    <main>
      <!-- Foods -->
      <section class="section foods">
        <h2>전통 음식</h2>
        <div class="cards">
          <article v-for="f in foods" :key="f.id" class="card">
            <img :src="f.img" :alt="f.name" class="card-image" />
            <h3>{{ f.name }}</h3>
            <p>{{ f.desc }}</p>
            <div class="card-actions">
              <button @click="openRecipe(f.id)" class="btn small">레시피</button>
              <button @click="downloadRecipeCard(f.id)" class="btn small ghost">카드 다운로드</button>
            </div>
            <!-- 송편 클릭으로 빚기 체험 -->
            <div v-if="f.id==='songpyeon'" class="songpyeon-maker">
              <p>송편의 탱글함을 느껴보세요 🍡</p>
              <canvas id="songpyeon-canvas" width="260" height="160"></canvas>
            </div>
          </article>
        </div>
      </section>

      <!-- Games -->
      <section class="section games">
        <h2>전통 놀이</h2>
        <div class="games-row">
          <div class="game">
            <h3>윷 던지기</h3>
            <button @click="rollYut" class="btn large">윷 던지기</button>
            <p v-if="yutResult">결과: <strong>{{ yutResult }}</strong></p>
          </div>

          <div class="game">
            <h3>강강술래</h3>
            <button @click="toggleGanggang" class="btn">{{ ganggangSpin ? '멈추기' : '원형 시작' }}</button>
            <div class="ganggang-container">
              <div :class="['circle-container', { spin: ganggangSpin }]">
                <div
                    v-for="i in 8"
                    :key="i"
                    class="dancer"
                    :style="{
          transform: `rotate(${(360 / 8) * i}deg) translate(100px) rotate(-${(360 / 8) * i}deg)`
        }"
                >
                  👩‍🌾
                </div>
                <div class="center-label"></div>
              </div>
            </div>
          </div>


          <!-- ✅ 제기차기 -->
          <div class="game">
            <h3>제기차기</h3>
            <canvas id="jegi-canvas" width="260" height="220" class="jegi-canvas"></canvas>
            <div class="jegi-controls">
              <button @click="kickJegi" class="btn small">차기!</button>
              <label class="pow">
                힘: <input type="range" min="6" max="16" step="1" value="10"
                          @input="setJegiPower(Number((($event.target as HTMLInputElement).value)))" />
              </label>
              <span class="score">성공: <strong>{{ jegiScore }}</strong></span>
            </div>
            <p class="muted">캔버스를 클릭/터치해도 튀어오릅니다.</p>
          </div>
        </div>
      </section>

      <!-- Stories -->
      <section class="section stories">
        <h2>가족의 달빛 일기</h2>
        <div class="story-uploader">
          <input type="file" accept="image/*" @change="handleStoryUpload" />
          <input v-model="storyText" placeholder="짧은 한줄 추억" />
          <button @click="$refs.fileInput?.click()" style="display:none">파일열기</button>
        </div>
        <div class="timeline">
          <article v-for="s in stories" :key="s.id" class="story-card">
            <img v-if="s.img" :src="s.img" alt="story image" class="story-thumb" @load="updateDominantFromLatestStory" />
            <div class="story-body">
              <div class="story-text">{{ s.text }}</div>
              <div class="story-date">{{ new Date(s.date).toLocaleString() }}</div>
            </div>
          </article>
        </div>
      </section>

      <!-- Gallery / Emotion -->
      <section class="section gallery">
        <h2>갤러리 & 감정의 파도</h2>
        <div class="gallery-grid">
          <img :src="IMG.gallery1" alt="gallery1" />
          <img :src="IMG.gallery2" alt="gallery2" />
          <img :src="IMG.songpyeon" alt="songpyeon" />
        </div>
        <canvas id="emotion-wave" class="emotion-wave" aria-hidden="true"></canvas>
      </section>

      <!-- Future -->
      <section class="section future">
        <h2>미래의 한가위</h2>
        <p>AI가 이야기하고, AR/VR과 연결되는 미래적 비전을 체험할 수 있습니다. (프로토타입 레벨)</p>
      </section>
    </main>

    <!-- footer -->
    <footer class="footer">
      <div>© 한가위 프로젝트 — Vue 3 + TypeScript + Vite</div>
      <div class="footer-actions">
        <button @click="toggleBgm" class="btn small">{{ bgPlaying ? '음악 끄기' : '음악 켜기' }}</button>
      </div>
    </footer>

    <!-- Recipe modal -->
    <div v-if="recipeModal.open" class="modal" role="dialog" aria-modal="true">
      <div class="modal-inner">
        <!-- ✅ 닫기 버튼 (더 크고 상단 고정) -->
        <button class="modal-close" @click="closeRecipe" aria-label="닫기">✕</button>

        <div v-if="recipeModal.foodId" class="recipe-content">
          <h3>{{ foods.find(f=>f.id===recipeModal.foodId)!.name }}</h3>

          <!-- 줄바꿈 유지 -->
          <pre class="recipe-text">
{{ foods.find(f=>f.id===recipeModal.foodId)!.recipe }}
      </pre>

          <img
              :src="foods.find(f=>f.id===recipeModal.foodId)!.img"
              alt="recipe image"
              class="recipe-image"
          />
        </div>
      </div>
    </div>


  </div>
</template>

<style scoped>
:root{
  --bg1:#060816;
  --accent: #f7c46b;
  --muted: #bfc8d4;
}
.app-root{font-family: Inter, 'Noto Sans KR', system-ui; color:var(--muted); background:linear-gradient(180deg,var(--bg1) 0%, #04050a 100%); min-height:100vh; overflow-x:hidden;}
/* moon canvas */
.moon-canvas{position:fixed; inset:0; z-index:-2; width:100%; height:100vh; display:block}
/* hero */
.hero{padding:6rem 1.25rem 4rem; display:flex; justify-content:center; align-items:flex-start}
.hero-inner{max-width:980px; text-align:center}
.title{font-size:clamp(28px,5vw,56px); color:var(--accent); font-weight:800; letter-spacing:-0.02em}
.subtitle{margin-top:0.75rem; color:#dfefff}
/* inputs */
.hero-controls{margin-top:1.2rem; display:flex; gap:0.6rem; justify-content:center; flex-wrap:wrap}
.wish-input{padding:0.6rem 0.8rem; border-radius:10px; border:none; width:220px}
.btn{background:linear-gradient(90deg,#ffcc66,#ff9966); color:#000; padding:0.6rem 0.9rem; border-radius:10px; border:none; cursor:pointer}
.btn.ghost{background:transparent; color:#fff; border:1px solid rgba(255,255,255,0.06)}
.btn.small{padding:0.4rem 0.6rem; font-size:0.9rem}
.btn.large{padding:0.9rem 1.2rem; font-size:1.05rem}
.muted{opacity:0.85}

/* lantern canvas */
.lantern-canvas{position:absolute; left:0; right:0; bottom:0; height:300px; pointer-events:none}

/* sections */
.section{padding:4rem 1.5rem}
.cards{display:flex; gap:1rem; flex-wrap:wrap; justify-content:center}
.card{background:rgba(255,255,255,0.03); padding:1rem; border-radius:12px; width:280px; box-shadow:0 8px 30px rgba(2,6,23,0.6)}
.card-image{width:100%; height:160px; object-fit:cover; border-radius:8px}
.card-actions{display:flex; gap:0.6rem; margin-top:0.6rem}

/* games */
.games-row{display:flex; gap:2rem; flex-wrap:wrap; justify-content:center}
.game{display:flex; flex-direction:column; align-items:center; gap:0.5rem; max-width:320px}
.game-thumb{width:220px; height:140px; object-fit:cover; border-radius:8px}
.circle-dance{display:flex; gap:12px; justify-content:center; align-items:center; margin-top:10px;}
.circle-dance.spin{animation:spin 6s linear infinite}
@keyframes spin { from{ transform: rotate(0deg)} to{ transform: rotate(360deg)}}

/* 제기차기 */
.jegi-canvas{background:radial-gradient(#0b0e18, #02030a); border-radius:12px; margin-top:0.25rem; box-shadow:0 0 10px rgba(255,255,255,0.08); width:260px; height:220px;}
.jegi-controls{display:flex; align-items:center; gap:10px; margin-top:6px; flex-wrap:wrap; justify-content:center;}
.jegi-controls .pow{font-size:0.9rem; opacity:0.9}
.jegi-controls .score{font-size:0.95rem}

/* stories */
.timeline{display:flex; gap:1rem; overflow-x:auto; padding:1rem 0}
.story-card{min-width:320px; background:rgba(255,255,255,0.02); padding:0.8rem; border-radius:10px; display:flex; gap:0.8rem}
.story-thumb{width:120px; height:80px; object-fit:cover; border-radius:6px}
.story-body{flex:1}
.story-text{font-weight:600}
.story-date{font-size:0.8rem; color:#9fb0d6}

/* gallery */
.gallery-grid{display:grid; grid-template-columns:repeat(auto-fill,minmax(160px,1fr)); gap:8px; margin-top:0.8rem}
.gallery-grid img{width:100%; height:140px; object-fit:cover; border-radius:8px}

/* emotion wave canvas */
.emotion-wave{width:100%; height:140px; display:block; margin-top:1rem}

/* footer */
.footer{padding:2rem 1.5rem; text-align:center; color:#93a3bb}

/* modal */
.modal{position:fixed; inset:0; display:flex; align-items:center; justify-content:center; background:rgba(2,6,23,0.6); z-index:60}
.modal-inner{background:#fff; color:#111; padding:1.25rem; border-radius:10px; width:min(680px,95%); position:relative;}
.modal-close{position:absolute; right:12px; top:12px; border:none; background:transparent; font-size:18px; cursor:pointer}

@media (max-width:720px){
  .hero{padding:3rem 1rem}
  .card{width:100%}
  /* 윷판 같은 고정폭 요소로 인한 가로 스크롤 방지 */
  .app-root{overflow-x:hidden}
}
/* 강강술래 (원형 회전형 이모티콘) */
.ganggang-container {
  position: relative;
  width: 240px;
  height: 240px;
  margin-top: 1rem;
  transform: translateX(-20px); /* 🔹 원의 중심을 위로 이동 */
}

.circle-container {
  position: relative;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.circle-container.spin {
  animation: spinCircle 10s linear infinite;
}

@keyframes spinCircle {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.dancer {
  position: absolute;
  left: 50%;
  top: 50%;
  font-size: 1.8rem;
  transform-origin: center center;
}

.center-label {
  position: absolute;
  color: var(--accent);
  font-weight: 600;
  font-size: 1rem;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.recipe-text {
  white-space: pre-wrap;
  font-size: 0.95rem;
  line-height: 1.6;
  margin-top: 0.5rem;
  color: #222;
}
.modal-close {
  position: absolute;
  right: 16px;
  top: 14px;
  border: none;
  background: #000;
  color: #fff; /* 밝은 노란색 포인트 */
  font-size: 22px;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 0 8px rgba(255, 255, 255, 0.25);
  transition: all 0.25s ease;
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-close:hover {
  background: rgba(255, 204, 102, 0.9);
  color: #000;
  transform: scale(1.1);
}
</style>
