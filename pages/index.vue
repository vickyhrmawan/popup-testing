<template>
  <section class="container">
    <h1>Pop Up POC</h1>

    <div class="demo-controls">
      <h2>Popup Trigger Demos</h2>
      <p>Choose a trigger mode to test:</p>

      <div class="trigger-buttons">
        <button class="btn" :class="{ active: mode === 'exit-intent' }" @click="mode = 'exit-intent'">
          Exit Intent
        </button>
        <button class="btn" :class="{ active: mode === 'timed' }" @click="mode = 'timed'">
          Timed (3s)
        </button>
        <button class="btn" :class="{ active: mode === 'scroll' }" @click="mode = 'scroll'">
          Scroll (50%)
        </button>
        <button class="btn" :class="{ active: mode === 'auto-close' }" @click="setAutoClose">
          Auto-Close (3s)
        </button>
        <button class="btn" :class="{ active: mode === 'manual' }" @click="showManual">
          Manual
        </button>
      </div>

      <p class="hint" v-if="mode === 'exit-intent'">Move your mouse above the browser window to trigger the popup.</p>
      <p class="hint" v-if="mode === 'timed'">The popup will appear in 3 seconds...</p>
      <p class="hint" v-if="mode === 'scroll'">Scroll down to 50% of the page to trigger the popup.</p>
      <p class="hint" v-if="mode === 'auto-close'">Popup will open and automatically close after 3 seconds.</p>
    </div>

    <!-- Filler content for scroll testing -->
    <div class="filler" v-for="n in 8" :key="n">
      <h3>Section {{ n }}</h3>
      <p>Lorem ipsum dolor sit amet consectetur, adipisicing elit. Voluptates sed tenetur quas! Laudantium odio consectetur, quia dolore veritatis quae quam nobis aperiam saepe, modi quibusdam facere tenetur omnis earum molestiae.</p>
    </div>

    <!-- Popup component: re-mounts on mode change via :key -->
    <PicrellPopup
      :key="popupKey"
      :trigger="popupTrigger"
      :delay="3000"
      :scroll-percent="50"
      :auto-close="autoCloseMs"
      :force="true"
      :animation="'zoom'"
      ref="popup"
      @submit="onPopupSubmit"
      @shown="onShown"
      @closed="onClosed"
    />
  </section>
</template>

<script>
import PicrellPopup from '~/components/PicrellPopup.vue'

export default {
  components: { PicrellPopup },
  data() {
    return {
      mode: 'exit-intent',
      popupKey: 0,
    }
  },
  computed: {
    popupTrigger() {
      return this.mode === 'auto-close' ? 'manual' : this.mode
    },
    autoCloseMs() {
      return this.mode === 'auto-close' ? 3000 : 0
    },
  },
  watch: {
    mode() {
      this.popupKey++
    },
  },
  methods: {
    setAutoClose() {
      this.mode = 'auto-close'
      this.$nextTick(() => {
        this.$refs.popup.show()
      })
    },
    showManual() {
      this.mode = 'manual'
      this.$nextTick(() => {
        this.$refs.popup.show()
      })
    },
    onPopupSubmit(email) {
      console.log('Email captured:', email)
    },
    onShown() {
      console.log('Popup shown')
    },
    onClosed() {
      console.log('Popup closed')
    },
  },
}
</script>

<style scoped>
.container {
  max-width: 700px;
  margin: 0 auto;
  padding: 40px 20px;
}

h1 {
  font-family: 'Montserrat', sans-serif;
  font-size: 36px;
  margin-bottom: 8px;
}

.subheading {
  color: #666;
  margin-bottom: 40px;
}

.demo-controls {
  background: #f8f9fa;
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 40px;
}

.demo-controls h2 {
  font-family: 'Montserrat', sans-serif;
  font-size: 20px;
  margin-bottom: 8px;
}

.trigger-buttons {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
  margin: 16px 0;
}

.btn {
  padding: 10px 20px;
  border: 2px solid #35495e;
  border-radius: 8px;
  background: #fff;
  color: #35495e;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}
.btn:hover {
  background: #35495e;
  color: #fff;
}
.btn.active {
  background: #e63946;
  border-color: #e63946;
  color: #fff;
}

.hint {
  color: #e63946;
  font-size: 14px;
  font-style: italic;
  margin-top: 8px;
}

.filler {
  margin-bottom: 32px;
  padding: 20px;
  background: #fafafa;
  border-radius: 8px;
}

.filler h3 {
  font-family: 'Montserrat', sans-serif;
  margin-bottom: 8px;
}

.filler p {
  color: #666;
  line-height: 1.6;
}
</style>
