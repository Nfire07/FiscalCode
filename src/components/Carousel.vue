<template>
  <div :class="['carousel-root', `carousel--${variant}`]">
    <Swiper
      v-if="variant === 'classic'"
      v-bind="classicOptions"
      :modules="classicModules"
      class="swiper-classic"
      @swiper="onSwiper"
    >
      <SwiperSlide v-for="(item, i) in items" :key="i">
        <div
          :class="['classic-card', { 'classic-card--clickable': modal }]"
          :tabindex="modal ? 0 : -1"
          :role="modal ? 'button' : undefined"
          :aria-label="modal ? `Open ${item.title}` : undefined"
          @click="modal && openModal(item)"
          @keydown.enter="modal && openModal(item)"
        >
          <div class="classic-card__img" :style="bgStyle(item)">
            <span v-if="!item.image" class="classic-card__emoji">{{ item.emoji }}</span>
          </div>
          <div class="classic-card__body">
            <span v-if="item.tag" class="classic-card__tag">{{ item.tag }}</span>
            <h3 class="classic-card__title">{{ item.title }}</h3>
            <p class="classic-card__desc">{{ item.description }}</p>
            <button v-if="modal" class="classic-card__cta" @click.stop="openModal(item)">{{ ctaLabel }}</button>
          </div>
        </div>
      </SwiperSlide>
    </Swiper>

    <div v-else-if="variant === 'hero'" class="hero-wrap" :style="{ height: heroHeight }">
      <Swiper
        v-bind="heroOptions"
        :modules="heroModules"
        class="swiper-hero"
        @swiper="onSwiper"
        @slide-change="onSlideChange"
      >
        <SwiperSlide v-for="(item, i) in items" :key="i">
          <div class="hero-slide" :style="heroBg(item)">
            <div class="hero-slide__overlay" />
            <div class="hero-slide__content">
              <span v-if="item.tag" class="hero-slide__chip">{{ item.tag }}</span>
              <h2 class="hero-slide__title">{{ item.title }}</h2>
              <p class="hero-slide__sub">{{ item.description }}</p>
              <div class="hero-slide__actions">
                <button v-if="modal" class="hero-btn" @click="openModal(item)">{{ ctaLabel }}</button>
                <slot name="hero-action" :item="item" />
              </div>
            </div>
          </div>
        </SwiperSlide>
      </Swiper>
      <div v-if="autoplay" class="hero-progress">
        <div class="hero-progress__bar" :style="{ animationDuration: `${autoplayDelay}ms` }" :key="activeIndex" />
      </div>
      <button class="hero-nav hero-nav--prev" aria-label="Precedente" @click="swiperInstance?.slidePrev()">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="15 18 9 12 15 6"/></svg>
      </button>
      <button class="hero-nav hero-nav--next" aria-label="Successivo" @click="swiperInstance?.slideNext()">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="9 18 15 12 9 6"/></svg>
      </button>
    </div>

    <div v-else-if="variant === 'fullscreen'" class="fullscreen-wrap">
      <Swiper
        v-bind="fullscreenOptions"
        :modules="fullscreenModules"
        class="swiper-fullscreen"
        @swiper="onSwiper"
      >
        <SwiperSlide v-for="(item, i) in items" :key="i">
          <div
            class="fullscreen-slide"
            :style="heroBg(item)"
            :tabindex="modal ? 0 : -1"
            @click="modal && openModal(item)"
            @keydown.enter="modal && openModal(item)"
          >
            <div class="fullscreen-slide__overlay" />
            <div class="fullscreen-slide__content">
              <h2 class="fullscreen-slide__title">{{ item.title }}</h2>
              <p v-if="!modal" class="fullscreen-slide__sub">{{ item.description }}</p>
              <p v-else class="fullscreen-slide__hint">Click to view more</p>
            </div>
            <div class="fullscreen-slide__counter">
              {{ String(i + 1).padStart(2, '0') }} / {{ String(items.length).padStart(2, '0') }}
            </div>
          </div>
        </SwiperSlide>
      </Swiper>
    </div>

    <Swiper
      v-else-if="variant === 'cards'"
      v-bind="cardsOptions"
      :modules="cardsModules"
      class="swiper-cards"
      @swiper="onSwiper"
    >
      <SwiperSlide v-for="(item, i) in items" :key="i">
        <div
          class="stack-card"
          :style="{ '--card-bg': item.color || 'var(--primary)' }"
          :tabindex="modal ? 0 : -1"
          @click="modal && openModal(item)"
          @keydown.enter="modal && openModal(item)"
        >
          <span class="stack-card__emoji">{{ item.emoji }}</span>
          <div class="stack-card__body">
            <span v-if="item.tag" class="stack-card__tag">{{ item.tag }}</span>
            <h3 class="stack-card__title">{{ item.title }}</h3>
            <p class="stack-card__desc">{{ item.description }}</p>
          </div>
          <div v-if="modal" class="stack-card__tap">{{ ctaLabel }} →</div>
        </div>
      </SwiperSlide>
    </Swiper>

    <Swiper
      v-else-if="variant === 'coverflow'"
      v-bind="coverflowOptions"
      :modules="coverflowModules"
      class="swiper-coverflow"
      @swiper="onSwiper"
    >
      <SwiperSlide v-for="(item, i) in items" :key="i">
        <div
          class="coverflow-card"
          :style="heroBg(item)"
          :tabindex="modal ? 0 : -1"
          @click="modal && openModal(item)"
          @keydown.enter="modal && openModal(item)"
        >
          <div class="coverflow-card__overlay" />
          <div class="coverflow-card__body">
            <span v-if="item.tag" class="coverflow-card__tag">{{ item.tag }}</span>
            <h3 class="coverflow-card__title">{{ item.title }}</h3>
          </div>
        </div>
      </SwiperSlide>
    </Swiper>

    <Teleport to="body">
      <Transition name="modal-fade">
        <div
          v-if="modalOpen && selectedItem"
          class="c-modal-backdrop"
          role="dialog"
          aria-modal="true"
          :aria-label="selectedItem.title"
          @click.self="closeModal"
          @keydown.esc="closeModal"
        >
          <div class="c-modal">
            <div
              class="c-modal__visual"
              :style="selectedItem.image
                ? { backgroundImage: `url(${selectedItem.image})`, backgroundSize: 'cover', backgroundPosition: 'center' }
                : { background: selectedItem.color || 'var(--primary)' }"
            >
              <span v-if="!selectedItem.image" class="c-modal__visual-emoji">{{ selectedItem.emoji }}</span>
              <button class="c-modal__close" aria-label="Close" @click="closeModal">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                  <line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/>
                </svg>
              </button>
            </div>
            <div class="c-modal__body">
              <span v-if="selectedItem.tag" class="c-modal__tag">{{ selectedItem.tag }}</span>
              <h2 class="c-modal__title">{{ selectedItem.title }}</h2>
              <p class="c-modal__text">{{ selectedItem.fullText || selectedItem.description }}</p>
              <div class="c-modal__footer">
                <slot name="modal-footer" :item="selectedItem" :close="closeModal">
                  <button class="c-modal__btn c-modal__btn--ghost" @click="closeModal">Close</button>
                  <button class="c-modal__btn c-modal__btn--primary" @click="$emit('cta-click', selectedItem); closeModal()">{{ ctaLabel }}</button>
                </slot>
              </div>
            </div>
          </div>
        </div>
      </Transition>
    </Teleport>
  </div>
</template>

<script>
import { Swiper, SwiperSlide } from 'swiper/vue'
import { Navigation, Pagination, Autoplay, EffectFade, EffectCards, EffectCoverflow, A11y } from 'swiper/modules'
import 'swiper/css'
import 'swiper/css/navigation'
import 'swiper/css/pagination'
import 'swiper/css/effect-fade'
import 'swiper/css/effect-cards'
import 'swiper/css/effect-coverflow'

export default {
  name: 'CustomCarousel',
  components: {
    Swiper,
    SwiperSlide
  },
  props: {
    items: {
      type: Array,
      required: true
    },
    variant: {
      type: String,
      default: 'classic',
      validator: v => ['classic', 'hero', 'fullscreen', 'cards', 'coverflow'].includes(v)
    },
    modal: {
      type: Boolean,
      default: true
    },
    autoplay: {
      type: Boolean,
      default: false
    },
    autoplayDelay: {
      type: Number,
      default: 4000
    },
    loop: {
      type: Boolean,
      default: true
    },
    ctaLabel: {
      type: String,
      default: 'Scopri di più'
    },
    heroHeight: {
      type: String,
      default: '520px'
    }
  },
  emits: ['modal-open', 'modal-close', 'cta-click'],
  data() {
    return {
      swiperInstance: null,
      modalOpen: false,
      selectedItem: null,
      activeIndex: 0,
      classicModules: [Navigation, Pagination, Autoplay, A11y],
      heroModules: [EffectFade, Pagination, Autoplay, A11y],
      fullscreenModules: [EffectFade, Autoplay, A11y],
      cardsModules: [EffectCards, Autoplay, A11y],
      coverflowModules: [EffectCoverflow, Pagination, Autoplay, A11y]
    }
  },
  computed: {
    autoplayCfg() {
      return this.autoplay ? { delay: this.autoplayDelay, disableOnInteraction: false } : false
    },
    classicOptions() {
      return {
        slidesPerView: 1.15,
        spaceBetween: 16,
        centeredSlides: true,
        loop: this.loop,
        autoplay: this.autoplayCfg,
        navigation: true,
        pagination: { clickable: true },
        grabCursor: true,
        breakpoints: {
          520: { slidesPerView: 2, centeredSlides: false },
          900: { slidesPerView: 3, centeredSlides: false }
        },
        a11y: { enabled: true }
      }
    },
    heroOptions() {
      return {
        effect: 'fade',
        fadeEffect: { crossFade: true },
        loop: this.loop,
        autoplay: this.autoplayCfg,
        speed: 800,
        pagination: { clickable: true },
        a11y: { enabled: true }
      }
    },
    fullscreenOptions() {
      return {
        effect: 'fade',
        fadeEffect: { crossFade: true },
        loop: this.loop,
        autoplay: this.autoplayCfg,
        speed: 1000,
        allowTouchMove: true,
        a11y: { enabled: true }
      }
    },
    cardsOptions() {
      return {
        effect: 'cards',
        grabCursor: true,
        loop: this.loop,
        autoplay: this.autoplayCfg,
        a11y: { enabled: true }
      }
    },
    coverflowOptions() {
      return {
        effect: 'coverflow',
        grabCursor: true,
        centeredSlides: true,
        slidesPerView: 'auto',
        loop: this.loop,
        autoplay: this.autoplayCfg,
        pagination: { clickable: true },
        coverflowEffect: { rotate: 40, stretch: 0, depth: 120, modifier: 1, slideShadows: true },
        a11y: { enabled: true }
      }
    }
  },
  methods: {
    onSwiper(s) {
      this.swiperInstance = s
    },
    onSlideChange(s) {
      this.activeIndex = s.realIndex
    },
    openModal(item) {
      this.selectedItem = item
      this.modalOpen = true
      if (this.swiperInstance && this.swiperInstance.autoplay) {
        this.swiperInstance.autoplay.stop()
      }
      this.$emit('modal-open', item)
    },
    closeModal() {
      this.$emit('modal-close', this.selectedItem)
      this.modalOpen = false
      this.selectedItem = null
      if (this.autoplay && this.swiperInstance && this.swiperInstance.autoplay) {
        this.swiperInstance.autoplay.start()
      }
    },
    bgStyle(item) {
      return item.image
        ? { backgroundImage: `url(${item.image})`, backgroundSize: 'cover', backgroundPosition: 'center' }
        : { background: item.color || 'var(--primary)' }
    },
    heroBg(item) {
      return item.image
        ? { backgroundImage: `url(${item.image})`, backgroundSize: 'cover', backgroundPosition: 'center' }
        : { background: item.color || 'var(--primary)' }
    }
  }
}
</script>

<style scoped>
.carousel-root {
  width: 100%;
  position: relative;
  color: var(--foreground);
}

.swiper-classic {
  padding: 1rem 0 3.5rem;
  --swiper-navigation-color: var(--primary);
  --swiper-pagination-color: var(--primary);
}

.classic-card {
  border-radius: 14px;
  overflow: hidden;
  background: var(--background);
  border: 1px solid color-mix(in srgb, var(--foreground) 10%, transparent);
  transition: transform .22s ease, box-shadow .22s ease;
}
.classic-card--clickable { cursor: pointer; }
.classic-card--clickable:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 32px color-mix(in srgb, var(--foreground) 12%, transparent);
}

.classic-card__img {
  width: 100%;
  height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.classic-card__emoji { font-size: 64px; }
.classic-card__body  { padding: 16px 18px 20px; }

.classic-card__tag {
  display: inline-block;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: .07em;
  text-transform: uppercase;
  padding: 3px 10px;
  border-radius: 99px;
  background: color-mix(in srgb, var(--primary) 12%, transparent);
  color: var(--primary);
  margin-bottom: 8px;
}
.classic-card__title {
  font-size: 16px;
  font-weight: 600;
  color: var(--foreground);
  margin: 0 0 6px;
}
.classic-card__desc {
  font-size: 13px;
  color: color-mix(in srgb, var(--foreground) 60%, transparent);
  line-height: 1.55;
  margin: 0 0 14px;
}
.classic-card__cta {
  background: transparent;
  border: 1.5px solid var(--foreground);
  padding: 7px 16px;
  border-radius: 99px;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  color: var(--foreground);
  transition: background .18s, color .18s;
}
.classic-card__cta:hover {
  background: var(--foreground);
  color: var(--background);
}

.hero-wrap {
  position: relative;
  border-radius: 0;
  overflow: hidden;
}
.swiper-hero {
  width: 100%;
  height: 100%;
  --swiper-pagination-color: var(--background);
  --swiper-pagination-bullet-inactive-color: color-mix(in srgb, var(--background) 50%, transparent);
  --swiper-pagination-bullet-inactive-opacity: 1;
}

.hero-slide {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: flex-end;
  position: relative;
}
.hero-slide__overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(to top, rgba(0,0,0,.72) 0%, rgba(0,0,0,.15) 55%, transparent 100%);
}
.hero-slide__content {
  position: relative;
  z-index: 2;
  padding: 36px 40px;
  width: 100%;
}
.hero-slide__chip {
  display: inline-block;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: .1em;
  text-transform: uppercase;
  padding: 4px 12px;
  border-radius: 99px;
  border: 1px solid rgba(255,255,255,.4);
  color: rgba(255,255,255,.9);
  margin-bottom: 10px;
  backdrop-filter: blur(4px);
}
.hero-slide__title {
  font-size: clamp(22px, 4vw, 36px);
  font-weight: 700;
  color: #fff;
  line-height: 1.15;
  margin: 0 0 8px;
  text-shadow: 0 2px 12px rgba(0,0,0,.3);
}
.hero-slide__sub {
  font-size: 15px;
  color: rgba(255,255,255,.78);
  line-height: 1.55;
  margin: 0 0 22px;
  max-width: 520px;
}
.hero-slide__actions { display: flex; gap: 10px; flex-wrap: wrap; }

.hero-btn {
  background: var(--background);
  color: var(--foreground);
  border: none;
  padding: 11px 24px;
  border-radius: 99px;
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
  transition: opacity .2s;
}
.hero-btn:hover { opacity: .85; }

.hero-nav {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  z-index: 10;
  width: 42px;
  height: 42px;
  border-radius: 50%;
  background: rgba(255,255,255,.15);
  border: 1px solid rgba(255,255,255,.3);
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  backdrop-filter: blur(6px);
  transition: background .2s;
}
.hero-nav:hover { background: rgba(255,255,255,.28); }
.hero-nav--prev { left: 16px; }
.hero-nav--next { right: 16px; }

.hero-progress {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 3px;
  background: color-mix(in srgb, var(--background) 25%, transparent);
  z-index: 10;
}
.hero-progress__bar {
  height: 100%;
  background: var(--background);
  animation: progress-fill linear forwards;
}
@keyframes progress-fill {
  from { width: 0%; }
  to   { width: 100%; }
}

.fullscreen-wrap { border-radius: 16px; overflow: hidden; }
.swiper-fullscreen { width: 100%; height: 92vh; max-height: 800px; }

.fullscreen-slide {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  cursor: pointer;
}
.fullscreen-slide__overlay {
  position: absolute;
  inset: 0;
  background: rgba(0,0,0,.38);
}
.fullscreen-slide__content {
  position: relative;
  z-index: 2;
  text-align: center;
  padding: 0 32px;
}
.fullscreen-slide__title {
  font-size: clamp(28px, 6vw, 64px);
  font-weight: 800;
  color: #fff;
  letter-spacing: -.02em;
  text-shadow: 0 4px 24px rgba(0,0,0,.4);
  margin: 0 0 12px;
}
.fullscreen-slide__sub {
  font-size: 18px;
  color: rgba(255,255,255,.8);
  max-width: 560px;
  margin: 0 auto;
  line-height: 1.6;
}
.fullscreen-slide__hint {
  font-size: 13px;
  color: rgba(255,255,255,.6);
  letter-spacing: .08em;
  text-transform: uppercase;
  margin: 0;
}
.fullscreen-slide__counter {
  position: absolute;
  bottom: 28px;
  right: 28px;
  font-size: 13px;
  font-weight: 600;
  color: rgba(255,255,255,.7);
  letter-spacing: .1em;
  z-index: 2;
}

.swiper-cards { width: 280px; height: 360px; margin: 2rem auto; }

.stack-card {
  width: 100%;
  height: 100%;
  border-radius: 20px;
  background: var(--card-bg);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 32px 28px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.stack-card__emoji {
  font-size: 72px;
  margin-bottom: 20px;
  display: block;
  filter: drop-shadow(0 4px 12px rgba(0,0,0,.3));
}
.stack-card__body { text-align: center; }
.stack-card__tag {
  display: inline-block;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: .1em;
  text-transform: uppercase;
  padding: 3px 10px;
  border-radius: 99px;
  border: 1px solid color-mix(in srgb, var(--secondary-foreground) 30%, transparent);
  color: color-mix(in srgb, var(--secondary-foreground) 80%, transparent);
  margin-bottom: 10px;
}
.stack-card__title {
  font-size: 22px;
  font-weight: 700;
  color: var(--secondary-foreground);
  margin: 0 0 8px;
}
.stack-card__desc {
  font-size: 13px;
  color: color-mix(in srgb, var(--secondary-foreground) 65%, transparent);
  line-height: 1.5;
  margin: 0;
}
.stack-card__tap {
  position: absolute;
  bottom: 18px;
  right: 18px;
  font-size: 11px;
  font-weight: 600;
  color: color-mix(in srgb, var(--secondary-foreground) 45%, transparent);
  letter-spacing: .06em;
}

.swiper-coverflow {
  padding: 2rem 0 3.5rem;
  --swiper-pagination-color: var(--primary);
}
.swiper-coverflow :deep(.swiper-slide) { width: 300px; height: 380px; }

.coverflow-card {
  width: 100%;
  height: 100%;
  border-radius: 16px;
  display: flex;
  align-items: flex-end;
  overflow: hidden;
  cursor: pointer;
  position: relative;
}
.coverflow-card__overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(to top, rgba(0,0,0,.7) 0%, transparent 60%);
}
.coverflow-card__body { position: relative; z-index: 2; padding: 18px 20px 22px; }
.coverflow-card__tag {
  display: block;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: .1em;
  text-transform: uppercase;
  color: rgba(255,255,255,.8);
  margin-bottom: 6px;
}
.coverflow-card__title { font-size: 18px; font-weight: 700; color: #fff; margin: 0; }

.c-modal-backdrop {
  position: fixed;
  inset: 0;
  background: color-mix(in srgb, var(--background) 95%, transparent);
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}
.c-modal {
  background: var(--background);
  border-radius: 18px;
  overflow: hidden;
  width: 100%;
  max-width: 500px;
  border: 1px solid color-mix(in srgb, var(--foreground) 8%, transparent);
}
.c-modal__visual {
  position: relative;
  height: 220px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.c-modal__visual-emoji { font-size: 80px; }
.c-modal__close {
  position: absolute;
  top: 14px;
  right: 14px;
  width: 34px;
  height: 34px;
  border-radius: 50%;
  background: rgba(0,0,0,.35);
  border: none;
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  backdrop-filter: blur(4px);
  transition: background .18s;
}
.c-modal__close:hover { background: rgba(0,0,0,.55); }
.c-modal__body { padding: 22px 26px 28px; }
.c-modal__tag {
  display: inline-block;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: .08em;
  text-transform: uppercase;
  padding: 3px 10px;
  border-radius: 99px;
  background: color-mix(in srgb, var(--primary) 12%, transparent);
  color: var(--primary);
  margin-bottom: 10px;
}
.c-modal__title { font-size: 22px; font-weight: 700; color: var(--foreground); margin: 0 0 10px; }
.c-modal__text {
  font-size: 14px;
  color: color-mix(in srgb, var(--foreground) 60%, transparent);
  line-height: 1.7;
  margin: 0 0 22px;
}
.c-modal__footer { display: flex; gap: 10px; justify-content: flex-end; }
.c-modal__btn {
  padding: 10px 22px;
  border-radius: 99px;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: all .18s;
}
.c-modal__btn--ghost {
  background: transparent;
  border: 1.5px solid color-mix(in srgb, var(--foreground) 20%, transparent);
  color: color-mix(in srgb, var(--foreground) 60%, transparent);
}
.c-modal__btn--ghost:hover {
  border-color: color-mix(in srgb, var(--foreground) 50%, transparent);
  color: var(--foreground);
}
.c-modal__btn--primary {
  background: var(--primary);
  border: 1.5px solid var(--primary);
  color: var(--primary-foreground);
}
.c-modal__btn--primary:hover { opacity: .88; }

.modal-fade-enter-active { animation: modal-pop-in .22s ease; }
.modal-fade-leave-active { animation: modal-pop-in .18s ease reverse; }
@keyframes modal-pop-in {
  from { opacity: 0; transform: scale(.94); }
  to   { opacity: 1; transform: scale(1); }
}
</style>