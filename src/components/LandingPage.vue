<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

type SocialLink = {
  label: string
  href: string
  hint?: string
  kind: 'twitch' | 'tg' | 'discord' | 'tiktok' | 'tg-personal'
}

const socials: SocialLink[] = [
  {
    label: 'Twitch',
    href: 'https://www.twitch.tv/kykar_',
    hint: 'Основной канал',
    kind: 'twitch',
  },
  {
    label: 'TG канал',
    href: 'https://t.me/kykartgk',
    hint: 'Анонсы и движ',
    kind: 'tg',
  },
  {
    label: 'Discord',
    href: 'https://discord.gg/n3s4aNfWev',
    hint: 'Комьюнити и войс',
    kind: 'discord',
  },
  {
    label: 'TikTok',
    href: 'https://www.tiktok.com/@kykar0',
    hint: 'Короткие клипы',
    kind: 'tiktok',
  },
  {
    label: 'TG личный',
    href: '',
    hint: 'Написать лично',
    kind: 'tg-personal',
  },
]

const twitchHref = 'https://www.twitch.tv/kykar_'

const rig = [
  { k: 'CPU', v: 'AMD Ryzen 5 5600', icon: 'cpu' },
  { k: 'GPU', v: 'GeForce RTX 3060 Ti', icon: 'gpu' },
  { k: 'ОЗУ', v: '32 GB', icon: 'ram' },
  { k: 'Монитор #1', v: '27" / 166 Hz', icon: 'monitor' },
  { k: 'Монитор #2', v: '32" / 166 Hz', icon: 'monitor' },
  { k: 'Микрофон', v: 'Fifine 8', icon: 'mic' },
  { k: 'Мышь', v: 'Logitech PRO Superlight', icon: 'mouse' },
  { k: 'Клавиатура', v: 'Attack Shark K86', icon: 'keyboard' },
  { k: 'Камера', v: 'iPhone 16', icon: 'camera' },
]

const contactTg = '@kykarrrr'
const year = new Date().getFullYear()

// Живой фон: положи файл в /public и оставь путь как есть:
// - public/arc-raiders-bg.mp4 (или .webm)
// - (опционально) public/arc-raiders-bg.jpg для poster
const bgVideoSrc = '/arc-raiders-bg.mp4'
const bgPosterSrc = '/arc-raiders-bg.jpg'

// Анимированная "иконка" (логотип) в шапке:
// положи файл в /public/arc-raiders-logo.mp4 (или .webm) и, опционально, poster.
const logoVideoSrc = '/arc-raiders-logo.mp4'
const logoPosterSrc = '/arc-raiders-logo.jpg'
const hasLogoVideo = ref(true)

// Фото стримера:
// - положи файл в /public под одним из имён ниже (любой формат подойдёт)
// - пока фото нет/путь битый, будет fallback /streamer-placeholder.svg
const streamerPhotoCandidates = [
  '/logo.jpeg',
  '/logo.jpg',
  '/logo.png',
  '/streamer.webp',
  '/streamer.png',
  '/streamer.jpg',
]
const streamerPhotoIdx = ref(0)
const streamerPhotoSrc = computed(() => streamerPhotoCandidates[streamerPhotoIdx.value] ?? streamerPhotoCandidates[0])
const streamerFallbackSrc = '/streamer-placeholder.svg'
const hasStreamerPhoto = ref(true)

function onStreamerPhotoError() {
  const next = streamerPhotoIdx.value + 1
  if (next < streamerPhotoCandidates.length) streamerPhotoIdx.value = next
  else hasStreamerPhoto.value = false
}

const contactHref = computed(() => `https://t.me/${contactTg.replace('@', '')}`)

const pageEl = ref<HTMLElement | null>(null)
const topSentinel = ref<HTMLElement | null>(null)
const showScrollTop = ref(false)
let revealIo: IntersectionObserver | null = null
let scrollIo: IntersectionObserver | null = null
let scrollRaf = 0

function setupReveal() {
  const root = pageEl.value
  if (!root || typeof IntersectionObserver === 'undefined') return

  const els = Array.from(root.querySelectorAll<HTMLElement>('[data-reveal]'))
  if (!els.length) return

  // Сначала сразу показываем то, что уже в зоне видимости (чтобы не было "пустого экрана").
  const vh = window.innerHeight || 0
  const toObserve: HTMLElement[] = []
  for (const el of els) {
    const r = el.getBoundingClientRect()
    if (r.top < vh * 0.92) el.classList.add('is-revealed')
    else toObserve.push(el)
  }

  // Включаем reveal-анимацию только когда JS готов (fallback: без этого классa всё видно).
  root.classList.add('reveal-enabled')

  if (!toObserve.length) return

  revealIo = new IntersectionObserver(
    (entries) => {
      for (const entry of entries) {
        if (!entry.isIntersecting) continue
        ;(entry.target as HTMLElement).classList.add('is-revealed')
        revealIo?.unobserve(entry.target)
      }
    },
    { threshold: 0.18, rootMargin: '0px 0px -10% 0px' },
  )

  for (const el of toObserve) revealIo.observe(el)
}

function getScrollTop() {
  const scrollingEl = document.scrollingElement as HTMLElement | null
  const docTop = Math.abs(document.documentElement.getBoundingClientRect().top || 0)
  const values = [
    window.scrollY || 0,
    window.pageYOffset || 0,
    scrollingEl?.scrollTop || 0,
    document.documentElement.scrollTop || 0,
    document.body.scrollTop || 0,
    pageEl.value?.scrollTop || 0,
    docTop,
  ]
  return Math.max(...values)
}

function onScroll() {
  showScrollTop.value = getScrollTop() > 520
}

function startScrollWatch() {
  const tick = () => {
    onScroll()
    scrollRaf = window.requestAnimationFrame(tick)
  }
  scrollRaf = window.requestAnimationFrame(tick)
}

function stopScrollWatch() {
  if (scrollRaf) {
    window.cancelAnimationFrame(scrollRaf)
    scrollRaf = 0
  }
}

function scrollToTop() {
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

onMounted(() => {
  if (typeof IntersectionObserver !== 'undefined' && topSentinel.value) {
    scrollIo = new IntersectionObserver((entries) => {
      const entry = entries[0]
      if (!entry) return
      showScrollTop.value = !entry.isIntersecting
    })
    scrollIo.observe(topSentinel.value)
  } else {
    onScroll()
    startScrollWatch()
    window.addEventListener('scroll', onScroll, { passive: true })
    document.addEventListener('scroll', onScroll, { passive: true })
  }
  setupReveal()
})
onBeforeUnmount(() => {
  scrollIo?.disconnect()
  stopScrollWatch()
  window.removeEventListener('scroll', onScroll)
  document.removeEventListener('scroll', onScroll)
  revealIo?.disconnect()
  revealIo = null
})
</script>

<template>
  <div ref="pageEl" class="page">
    <div class="bg" aria-hidden="true">
      <video
        class="bg__video"
        :src="bgVideoSrc"
        :poster="bgPosterSrc"
        autoplay
        muted
        loop
        playsinline
        preload="auto"
      ></video>
      <div class="bg__overlay"></div>
      <div class="bg__vignette"></div>
      <div class="bg__noise"></div>
    </div>

    <header class="header">
      <div class="container header__inner">
        <a class="brand" href="#top" aria-label="Kykar">
          <span class="brand__mark" aria-hidden="true">
            <video
              v-if="hasLogoVideo"
              class="brand__markVideo"
              :src="logoVideoSrc"
              :poster="logoPosterSrc"
              autoplay
              muted
              loop
              playsinline
              preload="auto"
              @error="hasLogoVideo = false"
            ></video>
            <span v-else class="brand__markFallback">K</span>
          </span>
          <span class="brand__text">
            <span class="brand__name">Kykar</span>
            <span class="brand__tag">стример • ARC RAIDERS</span>
          </span>
        </a>

        <nav class="nav" aria-label="Навигация">
          <a class="nav__link" href="#contacts">Контакты</a>
          <a class="nav__link" href="#setup">Оборудование</a>
          <a class="nav__link nav__link--donate" href="https://www.donationalerts.com/r/kykar_" target="_blank" rel="noreferrer">
            <span class="nav__linkIcon" aria-hidden="true">
              <svg viewBox="0 0 24 24" fill="none">
                <path
                  d="M6.5 6.5h4.8a6.2 6.2 0 1 1 0 12.4H6.5z"
                  stroke="currentColor"
                  stroke-width="1.8"
                  stroke-linejoin="round"
                />
                <path
                  d="M12.5 8.5l-2.2 4.1h2.2l-1.6 3.4"
                  stroke="currentColor"
                  stroke-width="1.6"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                />
              </svg>
            </span>
            Донат
          </a>
          <a class="nav__link nav__link--cta" :href="twitchHref" target="_blank" rel="noreferrer">
            Перейти на стрим
            <span class="nav__linkArrow" aria-hidden="true">
              <svg viewBox="0 0 24 24" fill="none">
                <path d="M6 12h12M14 6l4 6-4 6" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" />
              </svg>
            </span>
          </a>
        </nav>
      </div>
    </header>

    <main id="top" class="main">
      <div ref="topSentinel" class="scrollTopSentinel" aria-hidden="true"></div>
      <section class="hero">
        <div class="container hero__grid">
          <div class="hero__left" data-reveal :style="{ '--reveal-delay': '0ms' }">
            <h1 class="title title--oneLine">
              Я Никита, <span class="title__nowrap">можно просто&nbsp;<span class="title__accent">Кук</span></span>.
            </h1>
            <div class="hero__about">
              <div class="hero__aboutText">
                <p class="subtitle">Я из Бобруйска — стримлю, общаюсь и делаю честный движ без лишнего шума.</p>
                <p class="muted">По образованию повар, по жизни — военнослужащий.</p>
              </div>
              <div class="hero__aboutList">
                <div class="hero__aboutLabel">Коротко о стриме</div>
                <ul class="hero__aboutTags">
                  <li class="hero__aboutTag">
                    <span class="hero__aboutIcon" aria-hidden="true">
                      <svg viewBox="0 0 24 24" fill="none">
                        <path
                          d="M6 12a6 6 0 0 1 12 0c0 3-2.7 5.5-6 5.5S6 15 6 12Z"
                          stroke="currentColor"
                          stroke-width="1.6"
                        />
                        <path d="M9 12h6M10.5 14.5h3" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" />
                      </svg>
                    </span>
                    <span>Ламповое общение и катки, без токсика.</span>
                  </li>
                  <li class="hero__aboutTag">
                    <span class="hero__aboutIcon" aria-hidden="true">
                      <svg viewBox="0 0 24 24" fill="none">
                        <path
                          d="M8 12c2.5-3 5.5-3 8 0"
                          stroke="currentColor"
                          stroke-width="1.6"
                          stroke-linecap="round"
                        />
                        <path
                          d="M4.5 13.5c4-5 11-5 15 0"
                          stroke="currentColor"
                          stroke-width="1.6"
                          stroke-linecap="round"
                        />
                        <path
                          d="M12 18l1.2-1.2a2 2 0 1 0-2.8-2.8L9 15.4"
                          stroke="currentColor"
                          stroke-width="1.6"
                          stroke-linecap="round"
                        />
                      </svg>
                    </span>
                    <span>Комьюнити важнее хайпа — уважаем друг друга.</span>
                  </li>
                  <li class="hero__aboutTag">
                    <span class="hero__aboutIcon" aria-hidden="true">
                      <svg viewBox="0 0 24 24" fill="none">
                        <path
                          d="M7 7h10v10H7z"
                          stroke="currentColor"
                          stroke-width="1.6"
                          stroke-linejoin="round"
                        />
                        <path d="M9.5 12h5M12 9.5v5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" />
                      </svg>
                    </span>
                    <span>Люблю, когда всё по делу: от катки до разборов.</span>
                  </li>
                </ul>
              </div>
            </div>

          </div>

          <aside class="hero__right" data-reveal :style="{ '--reveal-delay': '120ms' }">
            <div class="hero__avatarCard">
              <div class="avatar avatar--hero" aria-label="Фото стримера">
                <img
                  v-if="hasStreamerPhoto"
                  class="avatar__img"
                  :src="streamerPhotoSrc"
                  alt="Фото стримера"
                  loading="lazy"
                  @error="onStreamerPhotoError"
                />
                <img
                  v-else
                  class="avatar__img"
                  :src="streamerFallbackSrc"
                  alt="Фото стримера"
                  loading="lazy"
                />
              </div>
              <div class="hero__caption">
                <div class="hero__name">Kykar_</div>
              </div>
            </div>

          </aside>
        </div>
      </section>

      <section id="contacts" class="section section--alt contacts">
        <div class="container">
          <div class="section__head">
            <h2 class="h2" data-reveal :style="{ '--reveal-delay': '0ms' }">Контакты</h2>
            <p class="muted contacts__hint" data-reveal :style="{ '--reveal-delay': '60ms' }">
              Все площадки — в одном месте.
            </p>
          </div>

          <div class="grid grid--5">
            <a
              v-for="(s, idx) in socials"
              :key="s.kind"
              class="socialCard"
              :class="`socialCard--${s.kind}`"
              :href="s.kind === 'tg-personal' ? contactHref : s.href"
              target="_blank"
              rel="noreferrer"
              data-reveal
              :style="{ '--reveal-delay': `${Math.min(idx, 8) * 50}ms` }"
            >
              <span class="socialCard__icon" aria-hidden="true">
                <svg v-if="s.kind === 'twitch'" viewBox="0 0 24 24" fill="none">
                  <path
                    d="M5 4h16v10l-4 4h-4l-2 2H9v-2H5V4Z"
                    stroke="currentColor"
                    stroke-width="1.6"
                    stroke-linejoin="round"
                  />
                  <path
                    d="M10 9v4M15 9v4"
                    stroke="currentColor"
                    stroke-width="1.6"
                    stroke-linecap="round"
                  />
                </svg>
                <svg v-else-if="s.kind === 'tg' || s.kind === 'tg-personal'" viewBox="0 0 24 24" fill="none">
                  <path
                    d="M20.4 5.3c.4-1.8-1.1-3-2.7-2.4L4.8 8.1c-1.7.7-1.7 3.1.1 3.7l3.3 1.1 1.3 4.1c.4 1.2 2 1.6 3 .7l1.9-1.8 3.5 2.6c1 .7 2.4.2 2.7-1.1l2.6-12.1Z"
                    stroke="currentColor"
                    stroke-width="1.6"
                    stroke-linejoin="round"
                  />
                  <path
                    d="M9.1 12.7 16.8 7.9c.5-.3 1 .3.6.7l-5.7 5.3c-.5.4-.8 1-.9 1.6l-.3 1.9"
                    stroke="currentColor"
                    stroke-width="1.6"
                    stroke-linecap="round"
                  />
                </svg>
                <svg v-else-if="s.kind === 'discord'" viewBox="0 0 24 24" fill="none">
                  <path
                    d="M8 7c2.7-1.6 5.3-1.6 8 0l1.4-.8c.9 1.4 1.5 3 1.6 4.7.2 2.9-.8 5.7-2.7 8.1-1.3.7-2.7 1.1-4.3 1.2L12 19l-1 .2c-1.6 0-3-.5-4.3-1.2-1.9-2.4-2.9-5.2-2.7-8.1.1-1.7.7-3.3 1.6-4.7L8 7Z"
                    stroke="currentColor"
                    stroke-width="1.6"
                    stroke-linejoin="round"
                  />
                  <path
                    d="M9.5 13.2h0M14.5 13.2h0"
                    stroke="currentColor"
                    stroke-width="2.4"
                    stroke-linecap="round"
                  />
                </svg>
                <svg v-else-if="s.kind === 'tiktok'" viewBox="0 0 24 24" fill="none">
                  <path
                    d="M14 4v10.2a4.2 4.2 0 1 1-3-4V7.2"
                    stroke="currentColor"
                    stroke-width="1.6"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                  />
                  <path
                    d="M14 6c1.2 1.8 3 3 5 3.2"
                    stroke="currentColor"
                    stroke-width="1.6"
                    stroke-linecap="round"
                  />
                </svg>
              </span>

              <span class="socialCard__text">
                <span class="socialCard__label">{{ s.label }}</span>
                <span v-if="s.hint" class="socialCard__hint">{{ s.hint }}</span>
              </span>
            </a>
          </div>
        </div>
      </section>

      <section id="setup" class="section setup">
        <div class="container">
          <div class="section__head">
            <h2 class="h2" data-reveal :style="{ '--reveal-delay': '0ms' }">Оборудование</h2>
            <p class="muted" data-reveal :style="{ '--reveal-delay': '60ms' }">
              Железо и девайсы, на которых идёт стрим.
            </p>
          </div>

          <div class="grid grid--3">
            <div
              v-for="(item, idx) in rig"
              :key="item.k"
              class="panel panel--tight setupCard"
              data-reveal
              :style="{ '--reveal-delay': `${Math.min(idx, 8) * 50}ms` }"
            >
              <span class="setupCard__icon" aria-hidden="true">
                <svg v-if="item.icon === 'cpu'" viewBox="0 0 24 24" fill="none">
                  <rect x="7" y="7" width="10" height="10" rx="2" stroke="currentColor" stroke-width="1.6" />
                  <path
                    d="M9 3v2M15 3v2M9 19v2M15 19v2M3 9h2M3 15h2M19 9h2M19 15h2"
                    stroke="currentColor"
                    stroke-width="1.6"
                    stroke-linecap="round"
                  />
                </svg>
                <svg v-else-if="item.icon === 'gpu'" viewBox="0 0 24 24" fill="none">
                  <rect x="4" y="7" width="14" height="10" rx="2" stroke="currentColor" stroke-width="1.6" />
                  <path d="M18 10h2v4h-2" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" />
                  <circle cx="9" cy="12" r="2" stroke="currentColor" stroke-width="1.6" />
                  <circle cx="13.5" cy="12" r="2" stroke="currentColor" stroke-width="1.6" />
                </svg>
                <svg v-else-if="item.icon === 'ram'" viewBox="0 0 24 24" fill="none">
                  <rect x="4" y="9" width="16" height="6" rx="2" stroke="currentColor" stroke-width="1.6" />
                  <path d="M7 7v2M11 7v2M15 7v2M7 15v2M11 15v2M15 15v2" stroke="currentColor" stroke-width="1.6" />
                </svg>
                <svg v-else-if="item.icon === 'monitor'" viewBox="0 0 24 24" fill="none">
                  <rect x="4" y="6" width="16" height="10" rx="2" stroke="currentColor" stroke-width="1.6" />
                  <path d="M8 20h8M12 16v4" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" />
                </svg>
                <svg v-else-if="item.icon === 'mic'" viewBox="0 0 24 24" fill="none">
                  <rect x="9" y="4" width="6" height="10" rx="3" stroke="currentColor" stroke-width="1.6" />
                  <path d="M5 11a7 7 0 0 0 14 0M12 18v2" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" />
                </svg>
                <svg v-else-if="item.icon === 'mouse'" viewBox="0 0 24 24" fill="none">
                  <rect x="8" y="3" width="8" height="18" rx="4" stroke="currentColor" stroke-width="1.6" />
                  <path d="M12 6v4" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" />
                </svg>
                <svg v-else-if="item.icon === 'keyboard'" viewBox="0 0 24 24" fill="none">
                  <rect x="3" y="7" width="18" height="10" rx="2" stroke="currentColor" stroke-width="1.6" />
                  <path
                    d="M7 10h1M10 10h1M13 10h1M16 10h1M7 13h10"
                    stroke="currentColor"
                    stroke-width="1.6"
                    stroke-linecap="round"
                  />
                </svg>
                <svg v-else-if="item.icon === 'camera'" viewBox="0 0 24 24" fill="none">
                  <rect x="5" y="7" width="14" height="10" rx="2" stroke="currentColor" stroke-width="1.6" />
                  <circle cx="12" cy="12" r="2.5" stroke="currentColor" stroke-width="1.6" />
                  <path d="M9 7l1-2h4l1 2" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" />
                </svg>
              </span>
              <div class="kv">
                <div class="kv__k">{{ item.k }}</div>
                <div class="kv__v">{{ item.v }}</div>
              </div>
            </div>
          </div>
        </div>
      </section>
    </main>

    <footer class="footer">
      <div class="container footer__inner">
        <div class="muted">© {{ year }} Kykar_ • стримы на Twitch</div>
      </div>
    </footer>

    <button v-if="showScrollTop" class="scrollTop" type="button" aria-label="Наверх" @click="scrollToTop">
      <svg viewBox="0 0 24 24" fill="none">
        <path d="M6 14l6-6 6 6" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" />
      </svg>
    </button>

  </div>
</template>
