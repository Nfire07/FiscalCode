<script>
export default {
  name: 'LanguageChanger',
  data() {
    return {
      lang: 'en'
    }
  },
  mounted() {
    const saved = localStorage.getItem('lang')
    if (saved) this.lang = saved
  },
  methods: {
    setLang(l) {
      this.lang = l
      localStorage.setItem('lang', l)
      this.$emit('change', l)
    }
  }
}
</script>

<style scoped>
.lang-switcher {
  display: flex;
  padding: 4px;
  border-radius: 9999px;
  box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px, rgba(0, 0, 0, 0.3) 0px 7px 13px -3px, rgba(0, 0, 0, 0.2) 0px -3px 0px inset;
  gap: 4px;
}

.lang-btn {
  border: none;
  background: transparent;
  color: var(--foreground);
  font-weight: 600;
  font-size: 12px;
  letter-spacing: 0.1em;
  padding: 6px 10px;
  border-radius: 9999px;
  cursor: pointer;
}

.lang-btn:hover {
  color: #323232;
}

.lang-btn.active {
  background-color: rgba(128, 128, 128, 0.35);
  color: var(--foreground);
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
}

</style>

<template>
  <div class="lang-switcher" role="radiogroup">
    <button
      class="lang-btn"
      :class="{ active: lang === 'en' }"
      role="radio"
      :aria-checked="lang === 'en'"
      @click="setLang('en')"
    >
      EN
    </button>

    <button
      class="lang-btn"
      :class="{ active: lang === 'it' }"
      role="radio"
      :aria-checked="lang === 'it'"
      @click="setLang('it')"
    >
      IT
    </button>
  </div>
</template>