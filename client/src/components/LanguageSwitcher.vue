<template>
  <div class="relative">
    <button
      class="flex items-center gap-2 px-3 py-2 bg-white border border-slate-200 rounded-lg hover:bg-slate-50 hover:border-slate-300 transition-colors cursor-pointer"
      @click="toggleDropdown"
      @blur="handleBlur"
    >
      <svg
        class="w-4 h-4 text-slate-500"
        viewBox="0 0 20 20"
        fill="none"
      >
        <circle cx="10" cy="10" r="7.5" stroke="currentColor" stroke-width="1.5"/>
        <path d="M3 10H17" stroke="currentColor" stroke-width="1.5"/>
        <path d="M10 3C10 3 7.5 5.5 7.5 10C7.5 14.5 10 17 10 17" stroke="currentColor" stroke-width="1.5"/>
        <path d="M10 3C10 3 12.5 5.5 12.5 10C12.5 14.5 10 17 10 17" stroke="currentColor" stroke-width="1.5"/>
      </svg>
      <span class="text-sm font-medium text-slate-700">{{ localeName }}</span>
      <svg
        class="w-3 h-3 text-slate-400 transition-transform"
        :class="{ 'rotate-180': isDropdownOpen }"
        viewBox="0 0 16 16"
        fill="none"
      >
        <path d="M4 6L8 10L12 6" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
      </svg>
    </button>

    <div v-if="isDropdownOpen" class="absolute top-[calc(100%+0.5rem)] right-0 min-w-[160px] bg-white border border-slate-200 rounded-xl shadow-xl z-[1000] overflow-hidden py-1">
      <button
        v-for="locale in availableLocales"
        :key="locale"
        class="w-full flex items-center gap-3 px-4 py-2.5 text-sm font-medium transition-colors text-left cursor-pointer border-0 bg-transparent font-[inherit]"
        :class="currentLocale === locale ? 'text-indigo-600 bg-indigo-50' : 'text-slate-700 hover:bg-slate-50'"
        @mousedown.prevent="selectLanguage(locale)"
      >
        <span>{{ getLanguageName(locale) }}</span>
        <svg
          v-if="currentLocale === locale"
          class="w-4 h-4 text-indigo-600 ml-auto"
          viewBox="0 0 18 18"
          fill="none"
        >
          <path d="M4 9L7.5 12.5L14 6" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useI18n } from '../composables/useI18n'

const { currentLocale, setLocale, availableLocales, localeName } = useI18n()

const isDropdownOpen = ref(false)

const languageNames = {
  en: 'English',
  ja: '日本語'
}

const getLanguageName = (locale) => {
  return languageNames[locale] || locale
}

const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value
}

const handleBlur = () => {
  // Delay to allow mousedown events on dropdown items to fire first
  setTimeout(() => {
    isDropdownOpen.value = false
  }, 200)
}

const selectLanguage = (locale) => {
  setLocale(locale)
  isDropdownOpen.value = false
}
</script>
