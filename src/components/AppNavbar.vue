<script setup lang="ts">
import { ref, watch } from 'vue'
import { RouterLink, useRoute } from 'vue-router'
import { Icon } from '@iconify/vue'

const route = useRoute()
const isMobileMenuOpen = ref(false)

const navItems = [
  { to: '/', label: 'Trang chủ', icon: 'lucide:home' },
  { to: '/leaderboard', label: 'Xếp hạng', icon: 'lucide:trophy' },
  { to: '/bookmarks', label: 'Yêu thích', icon: 'lucide:heart' },
  { to: '/members', label: 'Thành viên', icon: 'lucide:users' },
]

watch(
  () => route.path,
  () => {
    isMobileMenuOpen.value = false
  },
)
</script>

<template>
  <nav
    class="fixed top-0 left-0 right-0 z-40 bg-bg-deep/80 backdrop-blur-md border-b border-border-default"
  >
    <div class="max-w-5xl mx-auto px-4 sm:px-6 flex items-center justify-between h-14">
      <!-- Brand -->
      <RouterLink to="/" class="font-display font-bold text-lg text-text-primary tracking-tight">
        J2TEAM Community
      </RouterLink>

      <!-- Desktop nav -->
      <div class="hidden md:flex items-center gap-1">
        <RouterLink
          v-for="item in navItems"
          :key="item.to"
          :to="item.to"
          class="flex items-center gap-1.5 px-3 py-2 font-display text-sm tracking-wide text-text-secondary transition-colors duration-200 hover:text-text-primary"
          exact-active-class="!text-accent-coral"
          active-class=""
        >
          <Icon :icon="item.icon" class="w-4 h-4" />
          {{ item.label }}
        </RouterLink>
      </div>

      <!-- Mobile hamburger -->
      <button
        class="md:hidden p-2 text-text-secondary transition-colors hover:text-text-primary"
        aria-label="Menu"
        :aria-expanded="isMobileMenuOpen"
        @click="isMobileMenuOpen = !isMobileMenuOpen"
      >
        <Icon :icon="isMobileMenuOpen ? 'lucide:x' : 'lucide:menu'" class="w-5 h-5" />
      </button>
    </div>

    <!-- Mobile menu -->
    <Transition
      enter-active-class="transition-all duration-200 ease-out"
      enter-from-class="opacity-0 -translate-y-2"
      enter-to-class="opacity-100 translate-y-0"
      leave-active-class="transition-all duration-150 ease-in"
      leave-from-class="opacity-100 translate-y-0"
      leave-to-class="opacity-0 -translate-y-2"
    >
      <div
        v-if="isMobileMenuOpen"
        class="md:hidden border-t border-border-default bg-bg-deep/95 backdrop-blur-md"
      >
        <div class="max-w-5xl mx-auto px-4 py-2 flex flex-col gap-0.5">
          <RouterLink
            v-for="item in navItems"
            :key="item.to"
            :to="item.to"
            class="flex items-center gap-3 px-4 py-3 rounded-lg font-display text-sm tracking-wide text-text-secondary transition-colors duration-200 hover:text-text-primary hover:bg-bg-surface"
            exact-active-class="!text-accent-coral bg-bg-surface"
            active-class=""
          >
            <Icon :icon="item.icon" class="w-4 h-4" />
            {{ item.label }}
          </RouterLink>
        </div>
      </div>
    </Transition>
  </nav>
</template>
