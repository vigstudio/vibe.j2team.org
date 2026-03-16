<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { Icon } from '@iconify/vue'
import { REPO_URL } from '@/data/constants'

const CACHE_KEY = 'vibe-star-count'
const CACHE_TTL = 5 * 60 * 1000 // 5 minutes

const stars = ref(0)
const loaded = ref(false)
const failed = ref(false)

const goal = computed(() => Math.ceil((stars.value + 1) / 100) * 100)
const progress = computed(() => Math.min((stars.value / goal.value) * 100, 100))
const prevMilestone = computed(() => goal.value - 100)

async function fetchStars() {
  // Check sessionStorage cache
  const cached = sessionStorage.getItem(CACHE_KEY)
  if (cached) {
    try {
      const { value, timestamp } = JSON.parse(cached)
      if (Date.now() - timestamp < CACHE_TTL) {
        stars.value = value
        loaded.value = true
        return
      }
    } catch {
      // ignore invalid cache
    }
  }

  try {
    const res = await fetch('https://img.shields.io/github/stars/J2TEAM/vibe.j2team.org.json')
    if (!res.ok) throw new Error('Failed to fetch')
    const data = await res.json()
    const count = parseInt(data.value, 10)
    if (Number.isNaN(count)) throw new Error('Invalid star count')

    stars.value = count
    loaded.value = true

    sessionStorage.setItem(CACHE_KEY, JSON.stringify({ value: count, timestamp: Date.now() }))
  } catch {
    failed.value = true
  }
}

onMounted(fetchStars)
</script>

<template>
  <section
    v-if="loaded && !failed"
    class="max-w-5xl mx-auto px-4 sm:px-6 -mt-4 mb-8 animate-fade-up"
  >
    <a
      :href="REPO_URL"
      target="_blank"
      rel="noopener noreferrer nofollow"
      class="group block border border-border-default bg-bg-surface p-5 sm:p-6 transition-all duration-300 hover:border-accent-coral/50"
    >
      <!-- Header row -->
      <div class="flex flex-wrap items-center justify-between gap-3 mb-4">
        <div class="flex items-center gap-2.5">
          <Icon icon="lucide:star" class="w-5 h-5 text-accent-amber" />
          <span class="font-display text-sm tracking-widest text-accent-amber uppercase">
            Mục tiêu Star
          </span>
        </div>
        <span
          class="inline-flex items-center gap-2 border border-accent-coral/30 bg-accent-coral/10 px-4 py-1.5 font-display text-sm font-semibold text-accent-coral tracking-wide transition-all duration-300 group-hover:bg-accent-coral group-hover:text-bg-deep"
        >
          <Icon icon="lucide:github" class="w-4 h-4" />
          Tặng một sao trên GitHub
        </span>
      </div>

      <!-- Star count display -->
      <div class="flex items-baseline gap-2 mb-3">
        <span class="font-display text-3xl sm:text-4xl font-bold text-accent-coral tabular-nums">
          {{ stars.toLocaleString() }}
        </span>
        <span class="text-text-dim text-sm">/</span>
        <span class="font-display text-lg font-semibold text-text-secondary tabular-nums">
          {{ goal.toLocaleString() }}
        </span>
        <span class="text-text-dim text-sm">stars</span>
      </div>

      <!-- Progress bar -->
      <div class="relative h-3">
        <!-- Track -->
        <div class="absolute inset-0 bg-bg-deep rounded-full overflow-hidden">
          <!-- Filled bar -->
          <div
            class="progress-fill absolute inset-y-0 left-0 rounded-full"
            :style="{ width: `${progress}%` }"
          >
            <!-- Animated stripes -->
            <div class="progress-stripes absolute inset-0 rounded-full" />
            <!-- Shimmer sweep -->
            <div class="progress-shimmer absolute inset-0 rounded-full" />
          </div>
        </div>
        <!-- Glow dot at the tip (outside overflow-hidden so glow is visible) -->
        <div
          class="progress-tip absolute top-1/2 -translate-y-1/2 w-3 h-3 rounded-full bg-white transition-all duration-1000 ease-out"
          :style="{ left: `calc(${progress}% - 10px)` }"
        />
      </div>

      <!-- Milestones -->
      <div class="flex justify-between mt-2 text-xs text-text-dim font-display tabular-nums">
        <span>{{ prevMilestone.toLocaleString() }}</span>
        <span>{{ goal.toLocaleString() }}</span>
      </div>
    </a>
  </section>
</template>

<style scoped>
.progress-fill {
  background: linear-gradient(90deg, #ff6b4a, #ffb830);
  transition: width 1s ease-out;
}

/* Diagonal moving stripes */
.progress-stripes {
  background: repeating-linear-gradient(
    -45deg,
    transparent,
    transparent 6px,
    rgba(255, 255, 255, 0.1) 6px,
    rgba(255, 255, 255, 0.1) 12px
  );
  background-size: 17px 17px;
  animation: stripes-move 0.6s linear infinite;
}

@keyframes stripes-move {
  0% {
    background-position: 0 0;
  }
  100% {
    background-position: 17px 0;
  }
}

/* Shimmer sweep across the bar */
.progress-shimmer {
  background: linear-gradient(
    105deg,
    transparent 35%,
    rgba(255, 255, 255, 0.25) 45%,
    rgba(255, 255, 255, 0.4) 50%,
    rgba(255, 255, 255, 0.25) 55%,
    transparent 65%
  );
  background-size: 200% 100%;
  animation: shimmer-sweep 2.5s ease-in-out infinite;
}

@keyframes shimmer-sweep {
  0% {
    background-position: 200% 0;
  }
  100% {
    background-position: -200% 0;
  }
}

/* Glow pulse on the tip dot */
.progress-tip {
  animation: tip-pulse 2s ease-in-out infinite;
}

@keyframes tip-pulse {
  0%,
  100% {
    box-shadow:
      0 0 4px 1px rgba(255, 184, 48, 0.5),
      0 0 8px 3px rgba(255, 107, 74, 0.3);
  }
  50% {
    box-shadow:
      0 0 8px 3px rgba(255, 184, 48, 0.7),
      0 0 16px 6px rgba(255, 107, 74, 0.4);
  }
}
</style>
