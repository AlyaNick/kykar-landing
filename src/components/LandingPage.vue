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

// Фото стримера:
// - положи файл в /public/streamer.jpg (и оставь путь ниже)
// - или поменяй на своё
// - пока фото нет, будет fallback /streamer-placeholder.svg
const streamerPhotoSrc = '/streamer.jpg'
const streamerFallbackSrc = '/streamer-placeholder.svg'
const hasStreamerPhoto = ref(true)

const isContactsOpen = ref(false)
const contactHref = computed(() => `https://t.me/${contactTg.replace('@', '')}`)

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

onMounted(() => window.addEventListener('keydown', onGlobalKeydown))
onBeforeUnmount(() => window.removeEventListener('keydown', onGlobalKeydown))
</script>

<template>
  <div class="page">
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
          <span class="brand__mark" aria-hidden="true">K</span>
          <span class="brand__text">
            <span class="brand__name">Kykar</span>
            <span class="brand__tag">стример • ARC RAIDERS</span>
          </span>
        </a>

        <nav class="nav" aria-label="Навигация">
          <a class="nav__link" href="#about">Обо мне</a>
          <a class="nav__link" href="#content">Контент</a>
          <a class="nav__link" href="#setup">Сетап</a>
          <button class="nav__link nav__link--btn" type="button" @click="openContacts">
            Контакты
          </button>
        </nav>
      </div>
    </header>

    <main id="top" class="main">
      <section class="hero">
        <div class="container hero__grid">
          <div class="hero__left">
            <p class="kicker">Доброго времени суток!</p>
            <h1 class="title">
              Я Никита, можно просто <span class="title__accent">Кук</span>.
            </h1>
            <p class="subtitle">
              Играю в <strong>ARC RAIDERS</strong> (предложения игр приветствуются). Залетай —
              будет лампово и по делу.
            </p>

            <div class="cta">
              <a class="btn btn--primary" :href="twitchHref" target="_blank" rel="noreferrer">
                Смотреть стрим
              </a>
              <button class="btn btn--ghost" type="button" @click="openContacts">
                Связаться
              </button>
            </div>
          </div>

          <aside class="hero__right">
            <div class="card card--glow">
              <div class="card__top">
                <div class="avatar" aria-hidden="true">
                  <img
                    v-if="hasStreamerPhoto"
                    class="avatar__img"
                    :src="streamerPhotoSrc"
                    alt=""
                    loading="lazy"
                    @error="hasStreamerPhoto = false"
                  />
                  <img
                    v-else
                    class="avatar__img"
                    :src="streamerFallbackSrc"
                    alt=""
                    loading="lazy"
                  />
                </div>
                <div>
                  <div class="card__title">Kykar_</div>
                  <div class="card__meta">Беларусь • Бобруйск</div>
                </div>
              </div>
              <div class="divider" role="presentation"></div>
              <div class="muted">
                Если ищешь спокойный, честный стрим без токсика — залетай. 👊
              </div>
            </div>
          </aside>
        </div>
      </section>

      <section id="about" class="section">
        <div class="container">
          <h2 class="h2">Обо мне</h2>
          <div class="grid grid--2">
            <div class="panel">
              <p>Меня зовут Никита или просто Кук — рад приветствовать на своём канале!</p>
              <p class="muted">
                Сам Белорус из Бобруйска. По образованию повар, по факту военнослужащий,
                хобби — стриминг.
              </p>
              <ul class="bullets">
                <li>По образованию повар, по факту военнослужащий.</li>
                <li>Хобби — стриминг и комьюнити.</li>
                <li>Люблю честный движ без токсика.</li>
              </ul>
            </div>
            <div class="panel panel--outline">
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
          <h2 class="h2">Контент</h2>
          <div class="grid grid--3">
            <div class="panel">
              <h3 class="h3">Стримы</h3>
              <p class="muted">ARC RAIDERS + живое общение. Залетай на Twitch — там база.</p>
            </div>
            <div class="panel">
              <h3 class="h3">Комьюнити</h3>
              <p class="muted">Telegram для анонсов, Discord для войса и совместных каток.</p>
            </div>
            <div class="panel">
              <h3 class="h3">Клипы</h3>
              <p class="muted">TikTok — моменты, фейлы и лучшие фраги, чтобы не теряться.</p>
            </div>
          </div>
        </div>
      </section>

      <section id="setup" class="section">
        <div class="container">
          <div class="section__head">
            <h2 class="h2">Сетап</h2>
            <p class="muted">Железо и девайсы, на которых идёт стрим.</p>
          </div>

          <div class="grid grid--3">
            <div v-for="item in rig" :key="item.k" class="panel panel--tight">
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
        <div class="muted">© {{ year }} Kykar</div>
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
