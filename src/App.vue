<script setup>
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue'

const language = ref('nl')
const activeGame = ref('papa')
const rulesSection = ref(null)
const rulesVisible = ref(false)
const rulesHighlight = ref(false)
const scrollingToRules = ref(false)
const scrollProgress = ref(0)
const savedTheme = localStorage.getItem('theme')
const darkMode = ref(savedTheme ? savedTheme === 'dark' : window.matchMedia('(prefers-color-scheme: dark)').matches)
let rulesObserver
let rulesHighlightTimer
let scrollFrame

const boardLanes = [
  { color: 'red', points: [[125, 82], [125, 95], [125, 108]] },
  { color: 'blue', points: [[162.24, 146.5], [150.98, 140], [139.72, 133.5]] },
  { color: 'green', points: [[87.76, 146.5], [99.02, 140], [110.28, 133.5]] },
]

const startAreas = [
  { color: 'red', points: [[32, 36], [49, 36], [66, 36]] },
  { color: 'green', points: [[32, 214], [49, 214], [66, 214]] },
  { color: 'blue', points: [[184, 214], [201, 214], [218, 214]] },
]

const heroPawns = [
  { color: 'red', x: '71.806%', y: '27.130%', src: new URL('./assets/pawn-red.svg', import.meta.url).href },
  { color: 'green', x: '20.138%', y: '60.335%', src: new URL('./assets/pawn-green.svg', import.meta.url).href },
  { color: 'blue', x: '69.534%', y: '74.839%', src: new URL('./assets/pawn-blue.svg', import.meta.url).href },
]

const floatingPawns = [
  { color: 'red', position: 'float-red', angle: '-11deg', duration: '5.2s', delay: '-1.1s', distance: '-9px', src: new URL('./assets/pawn-red.svg', import.meta.url).href },
  { color: 'green', position: 'float-green', angle: '8deg', duration: '6.1s', delay: '-3.4s', distance: '-7px', src: new URL('./assets/pawn-green.svg', import.meta.url).href },
  { color: 'blue', position: 'float-blue', angle: '-6deg', duration: '5.7s', delay: '-2.2s', distance: '-10px', src: new URL('./assets/pawn-blue.svg', import.meta.url).href },
  { color: 'blue', position: 'float-blue-two', angle: '14deg', duration: '6.5s', delay: '-4.8s', distance: '-8px', src: new URL('./assets/pawn-blue.svg', import.meta.url).href },
  { color: 'red', position: 'float-red-two', angle: '5deg', duration: '5.9s', delay: '-.6s', distance: '-11px', src: new URL('./assets/pawn-red.svg', import.meta.url).href },
  { color: 'green', position: 'float-green-two', angle: '-15deg', duration: '6.8s', delay: '-5.2s', distance: '-6px', src: new URL('./assets/pawn-green.svg', import.meta.url).href },
  { color: 'green', position: 'float-green-three', angle: '12deg', duration: '5.5s', delay: '-2.9s', distance: '-9px', src: new URL('./assets/pawn-green.svg', import.meta.url).href },
  { color: 'blue', position: 'float-blue-three', angle: '-9deg', duration: '6.3s', delay: '-1.7s', distance: '-7px', src: new URL('./assets/pawn-blue.svg', import.meta.url).href },
  { color: 'red', position: 'float-red-three', angle: '17deg', duration: '5.4s', delay: '-4.1s', distance: '-10px', src: new URL('./assets/pawn-red.svg', import.meta.url).href },
]

const copy = {
  nl: {
    nav: {
      rules: 'Spelregels',
      switch: 'EN',
      flag: '🇬🇧',
      switchLabel: 'Switch to English',
      themeLabel: 'Schakel tussen lichte en donkere modus',
    },
    hero: {
      eyebrow: 'Speciaal gemaakt voor Vaderdag',
      title: 'Papa, erger je niet!',
      text: 'Twee bekende spellen, een uniek bord en vooral veel plezier samen.',
      cta: 'Bekijk de spelregels',
    },
    choose: 'Kies een spel',
    players: '3 spelers',
    quick: 'Snel uitgelegd',
    papa: {
      tab: 'Papa, erger je niet!',
      intro: 'Breng als eerste al je pionnen veilig naar je eigen eindvak.',
      facts: [
        ['Spelers', 'Precies 3'],
        ['Nodig', 'Uniek bord, pionnen en dobbelsteen'],
        ['Doel', 'Al je pionnen als eerste thuis'],
      ],
      sections: [
        {
          title: 'Klaarzetten',
          text: 'Iedere speler kiest een kleur en zet alle pionnen in zijn startvak. De jongste speler begint.',
        },
        {
          title: 'Zo speel je',
          text: 'Gooi om de beurt met de dobbelsteen en verplaats één pion evenveel vakjes vooruit.',
        },
        {
          title: 'Kies een richting',
          text: 'Spreek voor het begin samen af of iedereen met de klok mee of tegen de klok in speelt.',
        },
        {
          title: 'Een pion starten',
          text: 'Gooi je een 6? Dan mag je een nieuwe pion op het bord zetten. Daarna mag je nog een keer gooien.',
        },
        {
          title: 'Iemand terugsturen',
          text: 'Land je op een pion van een andere speler? Dan moet die pion terug naar zijn startvak.',
        },
        {
          title: 'Naar huis',
          text: 'Na een volledige ronde ga je je eigen eindstrook in. Je moet precies gooien om een pion thuis te krijgen.',
        },
      ],
      win: 'De eerste speler die alle pionnen thuis heeft, wint!',
    },
    connect: {
      tab: 'Vier op een rij',
      intro: 'Maak als eerste een rechte lijn van vier schijfjes op het bord van 5 × 5.',
      facts: [
        ['Spelers', '2 spelers'],
        ['Bord', 'Een raster van 5 × 5'],
        ['Doel', 'Vier op een rij'],
      ],
      sections: [
        {
          title: 'Klaarzetten',
          text: 'Iedere speler kiest een kleur. Begin met een leeg bord van 5 bij 5 vakjes.',
        },
        {
          title: 'Zo speel je',
          text: 'Leg om de beurt één schijfje van je eigen kleur in een leeg vakje.',
        },
        {
          title: 'Maak een rij',
          text: 'Probeer vier van jouw schijfjes naast elkaar te krijgen: horizontaal, verticaal of diagonaal.',
        },
        {
          title: 'Blokkeer elkaar',
          text: 'Let ook op de rij van de andere speler en blokkeer die op tijd.',
        },
      ],
      win: 'De eerste speler met vier schijfjes op een rechte lijn wint!',
    },
    footer: "Gemaakt voor en door de madeliefjes klas van de leefschool 'De Uitvlinder'",
  },
  en: {
    nav: {
      rules: 'Game rules',
      switch: 'NL',
      flag: '🇳🇱',
      switchLabel: 'Schakel naar Nederlands',
      themeLabel: 'Switch between light and dark mode',
    },
    hero: {
      eyebrow: "Made especially for Father's Day",
      title: "Papa, don't get annoyed!",
      text: 'Two familiar games, one unique board, and lots of fun together.',
      cta: 'Read the game rules',
    },
    choose: 'Choose a game',
    players: '3 players',
    quick: 'The short version',
    papa: {
      tab: "Papa, don't get annoyed!",
      intro: 'Be the first player to bring all your pawns safely home.',
      facts: [
        ['Players', 'Exactly 3'],
        ['You need', 'Unique board, pawns and a die'],
        ['Goal', 'Bring all your pawns home first'],
      ],
      sections: [
        {
          title: 'Set up',
          text: 'Each player chooses a color and puts all pawns in their starting area. The youngest player starts.',
        },
        {
          title: 'Your turn',
          text: 'Take turns rolling the die and move one pawn forward by that number of spaces.',
        },
        {
          title: 'Choose a direction',
          text: 'Before starting, agree whether everyone moves clockwise or counterclockwise.',
        },
        {
          title: 'Starting a pawn',
          text: 'Roll a 6? You may put a new pawn on the board. Then roll again.',
        },
        {
          title: 'Sending someone back',
          text: "Land on another player's pawn? That pawn must return to its starting area.",
        },
        {
          title: 'Going home',
          text: 'After one full lap, enter your own finishing lane. You need the exact roll to bring a pawn home.',
        },
      ],
      win: 'The first player to bring every pawn home wins!',
    },
    connect: {
      tab: 'Four in a row',
      intro: 'Be the first to make a straight line of four pieces on the 5 × 5 board.',
      facts: [
        ['Players', '2 players'],
        ['Board', 'A 5 × 5 grid'],
        ['Goal', 'Four in a row'],
      ],
      sections: [
        {
          title: 'Set up',
          text: 'Each player chooses a color. Start with an empty board of 5 by 5 spaces.',
        },
        {
          title: 'Your turn',
          text: 'Take turns placing one piece of your color in any empty space.',
        },
        {
          title: 'Make a row',
          text: 'Get four of your pieces in a line: horizontally, vertically, or diagonally.',
        },
        {
          title: 'Block each other',
          text: "Watch the other player's row too, and block it before it is complete.",
        },
      ],
      win: 'The first player with four pieces in a straight line wins!',
    },
    footer: "Made by and for de madeliefjesklas from the school 'De Uitvlinder'",
  },
}

const t = computed(() => copy[language.value])
const game = computed(() => t.value[activeGame.value])

function toggleLanguage() {
  language.value = language.value === 'nl' ? 'en' : 'nl'
}

function scrollToRules() {
  cancelAnimationFrame(scrollFrame)
  scrollingToRules.value = true
  rulesVisible.value = true
  scrollProgress.value = 0

  const startY = window.scrollY
  const targetY = startY + rulesSection.value.getBoundingClientRect().top
  const distance = targetY - startY
  const duration = 1050
  const startTime = performance.now()

  const animateScroll = (now) => {
    const progress = Math.min((now - startTime) / duration, 1)
    const eased = progress < .5
      ? 4 * progress * progress * progress
      : 1 - Math.pow(-2 * progress + 2, 3) / 2

    scrollProgress.value = progress
    window.scrollTo(0, startY + distance * eased)

    if (progress < 1) {
      scrollFrame = requestAnimationFrame(animateScroll)
      return
    }

    scrollingToRules.value = false
    rulesHighlight.value = false
    history.replaceState(null, '', '#regels')
    requestAnimationFrame(() => {
      rulesHighlight.value = true
      clearTimeout(rulesHighlightTimer)
      rulesHighlightTimer = window.setTimeout(() => {
        rulesHighlight.value = false
      }, 1100)
    })
  }

  scrollFrame = requestAnimationFrame(animateScroll)
}

watch(language, (value) => {
  document.documentElement.lang = value
  document.title = value === 'nl' ? 'Papa, erger je niet!' : "Dad, don't get annoyed!"
})

watch(darkMode, (value) => {
  document.documentElement.dataset.theme = value ? 'dark' : 'light'
  localStorage.setItem('theme', value ? 'dark' : 'light')
}, { immediate: true })

onMounted(() => {
  rulesObserver = new IntersectionObserver(([entry]) => {
    if (entry.isIntersecting) {
      rulesVisible.value = true
      rulesObserver.disconnect()
    }
  }, { threshold: 0.12 })

  rulesObserver.observe(rulesSection.value)
})

onBeforeUnmount(() => {
  rulesObserver?.disconnect()
  cancelAnimationFrame(scrollFrame)
  clearTimeout(rulesHighlightTimer)
})
</script>

<template>
  <div
    class="scroll-transition"
    :class="{ active: scrollingToRules }"
    :style="{ '--scroll-progress': scrollProgress }"
    aria-hidden="true"
  >
    <span></span>
  </div>

  <header class="topbar">
    <a class="brand" href="#" aria-label="De maddeliefjes klas">
      <span class="brand-mark" aria-hidden="true">
        <i v-for="petal in 8" :key="petal"></i>
        <b></b>
      </span>
      <span>De maddeliefjes klas</span>
    </a>
    <nav>
      <a class="rules-link" :class="{ 'is-scrolling': scrollingToRules }" href="#regels" @click.prevent="scrollToRules">
        {{ t.nav.rules }}
        <span aria-hidden="true">↓</span>
      </a>
      <button class="icon-button" type="button" :aria-label="t.nav.themeLabel" @click="darkMode = !darkMode">
        <svg v-if="darkMode" viewBox="0 0 24 24" aria-hidden="true">
          <path d="M21 15.2A9 9 0 0 1 8.8 3 9 9 0 1 0 21 15.2Z" />
        </svg>
        <svg v-else viewBox="0 0 24 24" aria-hidden="true">
          <circle cx="12" cy="12" r="4" />
          <path d="M12 2v2M12 20v2M4.93 4.93l1.42 1.42M17.66 17.66l1.41 1.41M2 12h2M20 12h2M4.93 19.07l1.42-1.41M17.66 6.34l1.41-1.41" />
        </svg>
      </button>
      <button class="language-button" type="button" :aria-label="t.nav.switchLabel" @click="toggleLanguage">
        <span class="language-flag" aria-hidden="true">{{ t.nav.flag }}</span>
        {{ t.nav.switch }}
        <span aria-hidden="true">↗</span>
      </button>
    </nav>
  </header>

  <main>
    <section class="hero">
      <div class="hero-copy">
        <p class="eyebrow">{{ t.hero.eyebrow }}</p>
        <h1>{{ t.hero.title }}</h1>
        <p class="hero-text">{{ t.hero.text }}</p>
        <a class="primary-button" :class="{ 'is-scrolling': scrollingToRules }" href="#regels" @click.prevent="scrollToRules">
          {{ t.hero.cta }}
          <span class="cta-arrow" aria-hidden="true">↓</span>
        </a>
      </div>

      <div class="board-art" aria-hidden="true">
        <svg class="hero-board" viewBox="0 0 250 250">
          <circle class="board-halo" cx="125" cy="125" r="104" />
          <g class="outer-track">
            <circle
              v-for="index in 33"
              :key="index"
              :class="{ red: index === 1, blue: index === 12, green: index === 23 }"
              :cx="125 + 79 * Math.sin((index - 1) * 2 * Math.PI / 33)"
              :cy="125 - 79 * Math.cos((index - 1) * 2 * Math.PI / 33)"
              r="6.3"
            />
          </g>
          <g v-for="lane in boardLanes" :key="lane.color" class="home-lane" :class="lane.color">
            <circle v-for="point in lane.points" :key="point.join('-')" :cx="point[0]" :cy="point[1]" r="6" />
          </g>
          <circle class="finish" cx="125" cy="125" r="17" />
          <circle class="finish-dot red" cx="125" cy="116" r="3.2" />
          <circle class="finish-dot green" cx="117" cy="130" r="3.2" />
          <circle class="finish-dot blue" cx="133" cy="130" r="3.2" />
        </svg>
        <div class="board-pawns">
          <img
            v-for="pawn in heroPawns"
            :key="pawn.color"
            class="hero-pawn"
            :class="pawn.color"
            :style="{ '--pawn-x': pawn.x, '--pawn-y': pawn.y }"
            :src="pawn.src"
            alt=""
          />
        </div>
        <img
          v-for="pawn in floatingPawns"
          :key="pawn.position"
          class="floating-pawn"
          :class="[pawn.color, pawn.position]"
          :style="{ '--float-angle': pawn.angle, '--float-duration': pawn.duration, '--float-delay': pawn.delay, '--float-distance': pawn.distance }"
          :src="pawn.src"
          alt=""
        />
      </div>
    </section>

    <section id="regels" ref="rulesSection" class="rules" :class="{ 'is-visible': rulesVisible, 'is-highlighted': rulesHighlight }">
      <div class="section-heading">
        <p class="eyebrow">{{ t.quick }}</p>
        <h2>{{ t.choose }}</h2>
      </div>

      <div class="game-tabs" role="tablist">
        <button
          v-for="key in ['papa', 'connect']"
          :key="key"
          type="button"
          role="tab"
          :aria-selected="activeGame === key"
          :class="{ active: activeGame === key }"
          @click="activeGame = key"
        >
          <svg v-if="key === 'papa'" class="tab-icon pawn-tab-icon" viewBox="0 0 24 28" aria-hidden="true">
            <circle cx="12" cy="7" r="4.5" />
            <path d="M7.5 23c.5-6.5 2-10 4.5-10s4 3.5 4.5 10c.1 1.2-.8 2-2 2h-5c-1.2 0-2.1-.8-2-2Z" />
          </svg>
          <span v-else class="tab-icon connect-grid-icon" aria-hidden="true">
            <i v-for="cell in 25" :key="cell" :class="{ filled: [3, 7, 11, 15, 19].includes(cell) }"></i>
          </span>
          {{ t[key].tab }}
        </button>
      </div>

      <article :key="activeGame" class="game-card">
        <div class="game-intro">
          <div>
            <span class="game-number">0{{ activeGame === 'papa' ? '1' : '2' }}</span>
            <h2>{{ game.tab }}</h2>
            <p>{{ game.intro }}</p>
          </div>
          <div v-if="activeGame === 'connect'" class="mini-grid" aria-hidden="true">
            <span v-for="cell in 25" :key="cell" :class="{ filled: [3, 7, 11, 15].includes(cell), alt: [4, 8, 12].includes(cell) }"></span>
          </div>
          <svg v-else class="rules-board" viewBox="0 0 250 250" aria-hidden="true">
            <circle class="board-halo" cx="125" cy="125" r="104" />
            <g class="start-areas">
              <g v-for="area in startAreas" :key="area.color" :class="area.color">
                <rect
                  :x="area.color === 'blue' ? 174 : 16"
                  :y="area.color === 'red' ? 20 : 198"
                  width="60"
                  height="32"
                  rx="8"
                />
                <circle v-for="point in area.points" :key="point.join('-')" :cx="point[0]" :cy="point[1]" r="6.5" />
              </g>
            </g>
            <g class="outer-track">
              <circle
                v-for="index in 33"
                :key="index"
                :class="{ red: index === 1, blue: index === 12, green: index === 23 }"
                :cx="125 + 79 * Math.sin((index - 1) * 2 * Math.PI / 33)"
                :cy="125 - 79 * Math.cos((index - 1) * 2 * Math.PI / 33)"
                r="6.3"
              />
            </g>
            <g v-for="lane in boardLanes" :key="lane.color" class="home-lane" :class="lane.color">
              <circle v-for="point in lane.points" :key="point.join('-')" :cx="point[0]" :cy="point[1]" r="6" />
            </g>
            <circle class="finish" cx="125" cy="125" r="17" />
            <circle class="finish-dot red" cx="125" cy="116" r="3.2" />
            <circle class="finish-dot green" cx="117" cy="130" r="3.2" />
            <circle class="finish-dot blue" cx="133" cy="130" r="3.2" />
          </svg>
        </div>

        <dl class="facts">
          <div v-for="[label, value] in game.facts" :key="label">
            <dt>{{ label }}</dt>
            <dd>{{ value }}</dd>
          </div>
        </dl>

        <ol class="steps">
          <li v-for="(section, index) in game.sections" :key="section.title">
            <span class="step-number">{{ String(index + 1).padStart(2, '0') }}</span>
            <div>
              <h3>{{ section.title }}</h3>
              <p>{{ section.text }}</p>
            </div>
          </li>
        </ol>

        <div class="win-banner">
          <span aria-hidden="true">★</span>
          <strong>{{ game.win }}</strong>
        </div>
      </article>
    </section>
  </main>

  <footer>
    <span class="footer-mark" aria-hidden="true">♥</span>
    <p>{{ t.footer }}</p>
  </footer>
</template>
