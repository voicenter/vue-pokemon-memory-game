<template>
  <div class="modal-card modal-card-root" ref="root">
    <header class="modal-card-head has-background-danger">
      <p class="modal-card-title has-text-light">You got them all!!</p>
      <button class="button is-small is-black" type="button" v-if="isMobile" @click="close">
        <b-icon icon="close" />
      </button>
    </header>
    <section class="modal-card-body">
      <Pokeball v-if="!isMobile" />

      <div class="control canvas-content" ref="content">
        <img src="img/8bits-coach.png">
        <h3>יום הולדת שמח - בהאבה מהנכדים ....</h3>
        <hr />

        <div class="tags has-addons are-medium">
          <span class="tag is-black">
            <b-icon icon="bullseye-arrow" />
          </span>
          <span class="tag is-black">Level</span>
          <span class="tag is-warning">
            {{ levelCount }}
          </span>
          <span v-if="isEasyMode" class="tag is-warning">
            <b-icon icon="baby-buggy" type="is-danger" />
          </span>
          <span v-if="isRouletteMode" class="tag is-warning">
            <b-icon icon="shuffle-variant" type="is-danger" />
          </span>
        </div>

        <div class="tags has-addons are-medium">
          <span class="tag is-black">
            <b-icon icon="counter" />
          </span>
          <span class="tag is-black">Score</span>
          <span class="tag is-link">{{ score | number }}</span>
          <span class="tag is-link">
            <b-icon icon="chart-line-variant" />
          </span>
          <span class="tag is-link">{{ scoreAvg | number }}</span>
        </div>

        <div class="tags has-addons are-medium">
          <span class="tag is-black">
            <b-icon icon="clock" />
          </span>
          <span class="tag is-black">Time</span>
          <span class="tag is-info">{{ timer | secToTimeStr(false) }}</span>
          <span class="tag is-info">
            <b-icon icon="chart-line-variant" />
          </span>
          <span class="tag is-info">{{ timerAvg | number }}</span>
        </div>
        <div class="only-canvas has-text-link">
          <center>bit.ly/pokemon-memory</center>
        </div>
      </div>
    </section>
    <footer class="modal-card-foot">
      <button class="button" type="button" @click="reload">
        <b-icon icon="reload" /> <span>Restart</span>
      </button>
      <button class="button" type="button" @click="save">
        <b-icon icon="download" />
      </button>
    </footer>
  </div>
</template>

<script>
import Pokeball from './Pokeball.vue'
import { filter } from 'lodash-es'
import { mapState, mapGetters, mapActions, mapMutations } from 'vuex'
import { Toast } from 'buefy/dist/components/toast'
import html2canvas from 'html2canvas'
import download from 'downloadjs'

export default {
  name: 'FinishModal',
  components: { Pokeball },
  computed: {
    ...mapGetters(['levelCount']),
    ...mapState(['score', 'level', 'timer', 'isEasyMode', 'isRouletteMode', 'isMobile']),
    levelLabel () {
      return this.level * 2
    },
    gameMode () {
      return filter([
        this.isEasyMode ? 'easy' : 'normal',
        this.isRouletteMode ? 'roulette' : ''
      ], val => !!val)
        .join('-')
    },
    scoreAvg () {
      const { level, score } = this
      if (score <= 0 || level <= 0) {
        return 0
      }

      return score / level
    },
    timerAvg () {
      const { timer, score } = this
      if (score <= 0 || timer <= 0) {
        return 0
      }

      return score / timer
    }
  },
  methods: {
    ...mapActions(['loadPokemon']),
    ...mapMutations({
      setLoading: 'set/isLoading'
    }),
    close () {
      this.$parent.close()
    },
    reload () {
      this.loadPokemon()
      this.$nextTick(() => {
        this.close()
      })
    },
    async save () {
      const { levelLabel, gameMode, score } = this

      this.setLoading(true)

      const imageName = `pokemon-memory-game-${levelLabel}-${gameMode}-${score}.jpg`

      try {
        const canvas = await html2canvas(this.$refs.content, {
          logging: false,
          onclone: doc => {
            doc.querySelector('.canvas-content')
              .classList
              .add('canvas') // customize css
          }
        })

        const data = canvas.toDataURL('image/jpeg', 1.0)

        download(data, imageName, 'image/jpeg')

        Toast.open({
          type: 'is-link',
          message: 'Use this image to share your result as a pokémon trainer!',
          duration: 3000
        })
      } catch (e) {
        console.error(e) // eslint-disable-line
        Toast.open({
          type: 'is-danger',
          message: 'There was a failure :('
        })
      } finally {
        setTimeout(() => {
          this.setLoading(false)
        }, 500)
      }
    }
  },
  mounted () {
    this.$ga('set', 'dimension0', 'time')
    this.$ga('set', 'dimension1', 'easy mode')
    this.$ga('set', 'dimension2', 'roulette mode')

    this.$ga('set', 'metric0', 'time')
    this.$ga('set', 'metric1', 'easy mode')
    this.$ga('set', 'metric2', 'roulette mode')

    const { isEasyMode, level, score, time, isRouletteMode, gameMode } = this

    this.$ga('send', {
      hitType: 'event',
      eventCategory: 'game',
      eventAction: `finish-${gameMode}`,
      eventLabel: `Finish level ${level * 2}`,
      eventValue: score,
      metric0: time,
      metric1: isEasyMode ? 1 : 0,
      metric2: isRouletteMode ? 1 : 0,
      nonInteraction: true
    })
  }
}
</script>

<style scoped>
  .only-canvas {
    display: none;
  }
  .canvas .only-canvas {
    display: block;
  }
  .canvas .tags {
    padding: 0 0.5em;
  }
  h3 {
    font-size: 1.5rem;
  }
  .modal-card-title, h3 {
    text-align: center;
    font-family: monospace;
    font-weight: bold;
  }
  .modal-card-foot {
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .modal-card-root {
    max-width: 400px;
  }

  @media screen and (max-width: 768px) {
    .modal-card-root {
      max-width: initial;
    }
  }
</style>
