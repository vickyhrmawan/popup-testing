<template>
  <transition name="popup-fade">
    <div v-if="visible" class="popup-overlay" @click.self="close">
      <div class="popup-modal" :class="animationClass">
        <button class="popup-close" @click="close">&times;</button>
        <div v-if="autoClose > 0 && countdown > 0" class="popup-countdown">
          Closing in {{ countdown }}s
        </div>
        <div class="popup-content">
          <slot>
            <div class="popup-default">
              <h2 class="popup-title">Wait! Don't leave yet</h2>
              <p class="popup-text">Get <strong>20% off</strong> your first order when you sign up today.</p>
              <form class="popup-form" @submit.prevent="onSubmit">
                <input
                  v-model="email"
                  type="email"
                  placeholder="Enter your email"
                  class="popup-input"
                  required
                />
                <button type="submit" class="popup-btn">Claim My Discount</button>
              </form>
              <a href="#" class="popup-dismiss" @click.prevent="close">No thanks, I'll pay full price</a>
            </div>
          </slot>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
export default {
  name: 'PicrellPopup',
  props: {
    // Trigger modes: 'exit-intent', 'timed', 'scroll', 'timed-exit', 'manual'
    trigger: {
      type: String,
      default: 'exit-intent',
    },
    // Delay in ms for 'timed' trigger
    delay: {
      type: Number,
      default: 5000,
    },
    // Scroll percentage (0-100) for 'scroll' trigger
    scrollPercent: {
      type: Number,
      default: 50,
    },
    // Cookie name to suppress repeat popups
    cookieName: {
      type: String,
      default: 'picrell_popup_dismissed',
    },
    // How many days to suppress after dismiss
    suppressDays: {
      type: Number,
      default: 7,
    },
    // Animation style: 'slide-up', 'zoom', 'fade'
    animation: {
      type: String,
      default: 'zoom',
    },
    // Auto-close after N ms (0 = disabled)
    autoClose: {
      type: Number,
      default: 0,
    },
    // Allow showing even if previously dismissed (for testing)
    force: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      visible: false,
      email: '',
      triggered: false,
      exitArmed: false,
      countdown: 0,
    }
  },
  computed: {
    animationClass() {
      return `popup-anim-${this.animation}`
    },
  },
  mounted() {
    if (!this.force && this.isDismissed()) return

    switch (this.trigger) {
      case 'exit-intent':
        this.setupExitIntent()
        break
      case 'timed':
        this.setupTimed()
        break
      case 'scroll':
        this.setupScroll()
        break
      case 'manual':
        // controlled externally via show()
        break
    }
  },
  beforeDestroy() {
    document.removeEventListener('mouseout', this.handleMouseOut)
    window.removeEventListener('scroll', this.handleScroll)
    if (this._timer) clearTimeout(this._timer)
  },
  methods: {
    setupExitIntent() {
      document.addEventListener('mouseout', this.handleMouseOut)
    },
    handleMouseOut(e) {
      // Trigger when mouse leaves from the top of the viewport
      if (e.clientY <= 0 && !this.triggered) {
        this.show()
      }
    },
    setupTimed() {
      this._timer = setTimeout(() => {
        if (!this.triggered) this.show()
      }, this.delay)
    },
    setupScroll() {
      window.addEventListener('scroll', this.handleScroll)
    },
    handleScroll() {
      const scrollTop = window.scrollY || document.documentElement.scrollTop
      const docHeight = document.documentElement.scrollHeight - window.innerHeight
      const scrolled = (scrollTop / docHeight) * 100

      if (scrolled >= this.scrollPercent && !this.triggered) {
        this.show()
      }
    },
    show() {
      this.triggered = true
      this.visible = true
      document.body.style.overflow = 'hidden'
      this.$emit('shown')

      if (this.autoClose > 0) {
        this.countdown = Math.ceil(this.autoClose / 1000)
        this._countdownInterval = setInterval(() => {
          this.countdown--
          if (this.countdown <= 0) {
            clearInterval(this._countdownInterval)
          }
        }, 1000)
        this._autoCloseTimer = setTimeout(() => {
          this.close()
        }, this.autoClose)
      }
    },
    close() {
      this.visible = false
      document.body.style.overflow = ''
      if (this._autoCloseTimer) clearTimeout(this._autoCloseTimer)
      if (this._countdownInterval) clearInterval(this._countdownInterval)
      this.setCookie()
      this.$emit('closed')
    },
    onSubmit() {
      this.$emit('submit', this.email)
      alert(`Thanks! We'll send the discount to: ${this.email}`)
      this.close()
    },
    isDismissed() {
      if (process.server) return false
      return document.cookie.includes(`${this.cookieName}=1`)
    },
    setCookie() {
      const expires = new Date()
      expires.setDate(expires.getDate() + this.suppressDays)
      document.cookie = `${this.cookieName}=1; expires=${expires.toUTCString()}; path=/`
    },
  },
}
</script>

<style scoped>
.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 99999;
  backdrop-filter: blur(3px);
}

.popup-modal {
  background: #fff;
  border-radius: 12px;
  max-width: 480px;
  width: 90%;
  padding: 40px 32px;
  position: relative;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

/* Animations */
.popup-anim-zoom {
  animation: popupZoom 0.35s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
.popup-anim-slide-up {
  animation: popupSlideUp 0.4s ease-out;
}
.popup-anim-fade {
  animation: popupFadeIn 0.3s ease-out;
}

@keyframes popupZoom {
  from { transform: scale(0.7); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}
@keyframes popupSlideUp {
  from { transform: translateY(60px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}
@keyframes popupFadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

/* Overlay transition */
.popup-fade-enter-active { transition: opacity 0.3s; }
.popup-fade-leave-active { transition: opacity 0.2s; }
.popup-fade-enter,
.popup-fade-leave-to { opacity: 0; }

.popup-close {
  position: absolute;
  top: 12px;
  right: 16px;
  background: none;
  border: none;
  font-size: 28px;
  cursor: pointer;
  color: #999;
  line-height: 1;
  padding: 0;
  transition: color 0.2s;
}
.popup-close:hover {
  color: #333;
}

.popup-countdown {
  text-align: center;
  font-size: 13px;
  color: #999;
  margin-bottom: 8px;
}

.popup-default {
  text-align: center;
}

.popup-title {
  font-family: 'Montserrat', sans-serif;
  font-size: 24px;
  font-weight: 700;
  color: #1a1a2e;
  margin-bottom: 12px;
}

.popup-text {
  font-size: 16px;
  color: #555;
  margin-bottom: 24px;
  line-height: 1.5;
}

.popup-form {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 16px;
}

.popup-input {
  padding: 14px 16px;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 16px;
  outline: none;
  transition: border-color 0.2s;
}
.popup-input:focus {
  border-color: #e63946;
}

.popup-btn {
  padding: 14px;
  background: #e63946;
  color: #fff;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.2s;
  font-family: 'Montserrat', sans-serif;
}
.popup-btn:hover {
  background: #c62d3a;
}

.popup-dismiss {
  display: inline-block;
  color: #999;
  font-size: 13px;
  text-decoration: underline;
  margin-top: 4px;
}
.popup-dismiss:hover {
  color: #666;
}
</style>
