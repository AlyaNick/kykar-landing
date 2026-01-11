<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

type SocialLink = {
  label: string
  href: string
  hint?: string
  kind: 'primary' | 'twitch' | 'tg' | 'discord' | 'tiktok'
}

const socials: SocialLink[] = [
  {
    label: 'Смотреть на Twitch',
    href: 'https://www.twitch.tv/kykar_',
    hint: 'Основной канал',
    kind: 'twitch',
  },
  {
    label: 'Telegram',
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
]

const twitchHref = 'https://www.twitch.tv/kykar_'

const rig = [
  { k: 'CPU', v: 'AMD Ryzen 5 5600' },
  { k: 'GPU', v: 'GeForce RTX 3060 Ti' },
  { k: 'ОЗУ', v: '32 GB' },
  { k: 'Монитор #1', v: '27" / 166 Hz' },
  { k: 'Монитор #2', v: '32" / 166 Hz' },
  { k: 'Микрофон', v: 'Fifine 8' },
  { k: 'Мышь', v: 'Logitech PRO Superlight' },
  { k: 'Клавиатура', v: 'Attack Shark K86' },
  { k: 'Камера', v: 'iPhone 16' },
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

const isContactsOpen = ref(false)
const contactHref = computed(() => `https://t.me/${contactTg.replace('@', '')}`)

const pageEl = ref<HTMLElement | null>(null)
let revealIo: IntersectionObserver | null = null

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

function toggleContacts() {
  isContactsOpen.value = !isContactsOpen.value
}

function openContacts() {
  isContactsOpen.value = true
}

function closeContacts() {
  isContactsOpen.value = false
}

function onGlobalKeydown(e: KeyboardEvent) {
  if (e.key === 'Escape') closeContacts()
}

onMounted(() => {
  window.addEventListener('keydown', onGlobalKeydown)
  setupReveal()
})
onBeforeUnmount(() => {
  window.removeEventListener('keydown', onGlobalKeydown)
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
          <a class="nav__link" href="#about">Обо мне</a>
          <a class="nav__link" href="#content">Контент</a>
          <a class="nav__link" href="#setup">Оборудование</a>
          <button class="nav__link nav__link--btn" type="button" @click="openContacts">
            Контакты
          </button>
        </nav>
      </div>
    </header>

    <main id="top" class="main">
      <section class="hero">
        <div class="container hero__grid">
          <div class="hero__left" data-reveal :style="{ '--reveal-delay': '0ms' }">
            <p class="kicker">Доброго времени суток!</p>
            <h1 class="title title--oneLine">
              Я Никита, <span class="title__nowrap">можно просто&nbsp;<span class="title__accent">Кук</span></span>.
            </h1>
            <p class="subtitle">
              Играю в <strong>ARC RAIDERS</strong> (предложения игр приветствуются). Залетай —
              будет лампово и по делу.
            </p>

            <div class="hero__chips" aria-label="Темы">
              <span class="chip chip--brand">ARC RAIDERS</span>
              <span class="chip">живое общение</span>
              <span class="chip">без токсика</span>
            </div>

            <div class="cta">
              <a class="btn btn--primary" :href="twitchHref" target="_blank" rel="noreferrer">
                Смотреть стрим
              </a>
              <button class="btn btn--ghost" type="button" @click="openContacts">
                Связаться
              </button>
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
                <div class="hero__tag muted">стример • ARC RAIDERS</div>
              </div>
            </div>
          </aside>
        </div>
      </section>

      <section id="about" class="section">
        <div class="container">
          <h2 class="h2" data-reveal :style="{ '--reveal-delay': '0ms' }">Обо мне</h2>
          <div class="grid grid--2">
            <div class="panel" data-reveal :style="{ '--reveal-delay': '60ms' }">
              <p>
                Я Никита, можно просто <strong>Кук</strong>. Я из Бобруйска — стримлю, общаюсь и делаю
                честный движ без лишнего шума.
              </p>
              <p class="muted">По образованию повар, по жизни — военнослужащий. В онлайне — свой человек.</p>
              <ul class="bullets">
                <li>Ламповое общение и катки, без токсика.</li>
                <li>Комьюнити важнее хайпа — уважаем друг друга.</li>
                <li>Люблю, когда всё по делу: от катки до разборов.</li>
              </ul>
              <div class="about-note">
                <div class="about-note__row">
                  <span class="muted">Ник:</span>&nbsp;<strong>Kykar_</strong>
                </div>
                <div class="about-note__row">
                  <span class="muted">Город:</span>&nbsp;Беларусь • Бобруйск
                </div>
              </div>
            </div>
            <div class="panel panel--outline" data-reveal :style="{ '--reveal-delay': '120ms' }">
              <div class="stat">
                <div class="stat__k">Основная игра</div>
                <div class="stat__v">ARC RAIDERS</div>
              </div>
              <div class="stat">
                <div class="stat__k">Формат</div>
                <div class="stat__v">общение • катки • разбор</div>
              </div>
              <div class="stat">
                <div class="stat__k">Игры на будущее</div>
                <div class="stat__v">предлагай — обсудим</div>
              </div>
            </div>
          </div>
        </div>
      </section>

      <section id="content" class="section section--alt">
        <div class="container">
          <h2 class="h2" data-reveal :style="{ '--reveal-delay': '0ms' }">Контент</h2>
          <div class="grid grid--3">
            <div class="panel" data-reveal :style="{ '--reveal-delay': '60ms' }">
              <h3 class="h3">Стримы</h3>
              <p class="muted">ARC RAIDERS + живое общение. Залетай на Twitch — там база.</p>
            </div>
            <div class="panel" data-reveal :style="{ '--reveal-delay': '120ms' }">
              <h3 class="h3">Комьюнити</h3>
              <p class="muted">Telegram для анонсов, Discord для войса и совместных каток.</p>
            </div>
            <div class="panel" data-reveal :style="{ '--reveal-delay': '180ms' }">
              <h3 class="h3">Клипы</h3>
              <p class="muted">TikTok — моменты, фейлы и лучшие фраги, чтобы не теряться.</p>
            </div>
          </div>
        </div>
      </section>

      <section id="setup" class="section">
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
              class="panel panel--tight"
              data-reveal
              :style="{ '--reveal-delay': `${Math.min(idx, 8) * 50}ms` }"
            >
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

    <div v-if="isContactsOpen" class="fab-overlay" @click="closeContacts"></div>

    <div class="fab">
      <transition name="fab-pop">
        <div v-if="isContactsOpen" id="contact-menu" class="fab__menu" role="menu">
          <a
            class="fab__item fab__item--tg"
            :href="contactHref"
            target="_blank"
            rel="noreferrer"
            role="menuitem"
          >
            <span class="fab__icon" aria-hidden="true">
              <svg viewBox="0 0 24 24" fill="none">
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
            </span>
            <span class="fab__label">Написать в TG</span>
          </a>

          <a
            v-for="s in socials"
            :key="s.href"
            class="fab__item"
            :class="`fab__item--${s.kind}`"
            :href="s.href"
            target="_blank"
            rel="noreferrer"
            role="menuitem"
          >
            <span class="fab__icon" aria-hidden="true">
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
              <svg v-else-if="s.kind === 'tg'" viewBox="0 0 24 24" fill="none">
                <path
                  d="M20.4 5.3c.4-1.8-1.1-3-2.7-2.4L4.8 8.1c-1.7.7-1.7 3.1.1 3.7l3.3 1.1 1.3 4.1c.4 1.2 2 1.6 3 .7l1.9-1.8 3.5 2.6c1 .7 2.4.2 2.7-1.1l2.6-12.1Z"
                  stroke="currentColor"
                  stroke-width="1.6"
                  stroke-linejoin="round"
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
            <span class="fab__label">{{ s.label }}</span>
          </a>
        </div>
      </transition>

      <button
        class="fab__btn"
        type="button"
        aria-label="Контакты"
        aria-controls="contact-menu"
        :aria-expanded="isContactsOpen"
        @click="toggleContacts"
        @keydown.esc.prevent="closeContacts"
      >
        <span class="fab__btnIcon" aria-hidden="true">
          <svg v-if="!isContactsOpen" viewBox="0 0 24 24" fill="none">
            <path
              d="M7.5 7.5h9A3.5 3.5 0 0 1 20 11v1a3.5 3.5 0 0 1-3.5 3.5H12l-3.2 2.4c-.7.5-1.8 0-1.8-.9V15.5A3.5 3.5 0 0 1 3.5 12v-1A3.5 3.5 0 0 1 7 7.5Z"
              stroke="currentColor"
              stroke-width="1.8"
              stroke-linecap="round"
              stroke-linejoin="round"
            />
          </svg>
          <svg v-else viewBox="0 0 24 24" fill="none">
            <path
              d="M7 7l10 10M17 7L7 17"
              stroke="currentColor"
              stroke-width="1.8"
              stroke-linecap="round"
            />
          </svg>
        </span>
        <span class="fab__btnText">Контакты</span>
      </button>
    </div>
  </div>
</template>
