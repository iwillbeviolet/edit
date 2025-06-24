<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'

export type BookmarkType = {
  name: string
  chord: string
  url: string
  icon?: string // e.g. 'i-logos:github'
  color?: string
}

const props = defineProps<{ isInputGloballyFocused: boolean }>()

const currentChordInput = ref('')
const activePossibleChords = ref<BookmarkType[]>([])
let chordTimeout: NodeJS.Timeout | null = null

const initialBookmarksData: BookmarkType[] = [
  {
    name: 'Hacker News',
    chord: 'HN',
    url: 'https://news.ycombinator.com/',
    icon: 'i-logos:ycombinator'
  },
  {
    name: 'GitHub',
    chord: 'GH',
    url: 'https://github.com/',
    icon: 'i-simple-icons:github'
  },
  {
    name: 'Reddit',
    chord: 'RD',
    url: 'https://reddit.com/',
    icon: 'i-logos:reddit-icon'
  },
  {
    name: 'Twitter',
    chord: 'TW',
    url: 'https://twitter.com/',
    icon: 'i-logos:twitter'
  },
  {
    name: 'YouTube',
    chord: 'YT',
    url: 'https://youtube.com/',
    icon: 'i-logos:youtube-icon'
  },
  {
    name: "Beginner's Guide",
    chord: 'BG',
    url: '/beginners-guide',
    icon: 'i-lucide:book-open-text'
  },
  {
    name: 'Wikipedia',
    chord: 'WK',
    url: 'https://wikipedia.org/',
    icon: 'i-simple-icons:wikipedia'
  }
]

const resetChord = () => {
  currentChordInput.value = ''
  activePossibleChords.value = []
  if (chordTimeout) clearTimeout(chordTimeout)
  chordTimeout = null
}

const handleBookmarkClick = (bookmark: BookmarkType) => {
  window.open(bookmark.url, '_self')
}

const handleKeyDown = (e: KeyboardEvent) => {
  if (
    props.isInputGloballyFocused ||
    e.altKey ||
    e.metaKey ||
    e.ctrlKey ||
    e.shiftKey
  )
    return

  const active = document.activeElement as HTMLElement | null
  if (
    active?.tagName === 'INPUT' ||
    active?.tagName === 'TEXTAREA' ||
    active?.isContentEditable
  )
    return

  const key = e.key.toUpperCase()
  if (chordTimeout) clearTimeout(chordTimeout)

  if (!currentChordInput.value) {
    const matches = initialBookmarksData.filter((b) => b.chord.startsWith(key))
    if (matches.length) {
      e.preventDefault()
      currentChordInput.value = key
      activePossibleChords.value = matches
      chordTimeout = setTimeout(resetChord, 2000)
    }
  } else {
    const next = currentChordInput.value + key
    const match = activePossibleChords.value.find((b) => b.chord === next)
    if (match) {
      window.open(match.url, '_self')
      resetChord()
    } else {
      const filtered = initialBookmarksData.filter((b) =>
        b.chord.startsWith(next)
      )
      if (filtered.length) {
        currentChordInput.value = next
        activePossibleChords.value = filtered
        chordTimeout = setTimeout(resetChord, 2000)
      } else {
        resetChord()
      }
    }
  }
}

onMounted(() => document.addEventListener('keydown', handleKeyDown))
onUnmounted(() => {
  document.removeEventListener('keydown', handleKeyDown)
  if (chordTimeout) clearTimeout(chordTimeout)
})
</script>

<template>
  <div class="space-y-4">
    <div class="flex items-center justify-between">
      <div class="flex items-center gap-2 text-text-2">
        <i class="i-lucide:bookmark w-5 h-5" />
        <h2 class="text-xl">Bookmarks</h2>
      </div>
      <div v-if="currentChordInput" class="px-3 py-1 rounded-md text-sm font-medium bg-yellow-200/20 text-yellow-600">
        Chord: {{ currentChordInput }}...
      </div>
    </div>

    <div class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-5 xl:grid-cols-6 gap-2">
      <button v-for="bookmark in initialBookmarksData" :key="bookmark.name" :class="[
        'rounded-md border border-div bg-bg-alt px-3 py-2 flex justify-between items-center text-left transition-opacity duration-150 group',
        activePossibleChords.some((ab) => ab.chord === bookmark.chord)
          ? bookmark.chord === currentChordInput
            ? 'opacity-100 ring-2 ring-primary ring-offset-2 ring-offset-bg'
            : 'opacity-75'
          : currentChordInput
            ? 'opacity-30'
            : 'opacity-100'
      ]" @click="handleBookmarkClick(bookmark)">
        <div class="flex-1 min-w-0 flex gap-3 justify-between">
          <div class="font-medium truncate text-left flex items-center gap-2">
            <i v-if="bookmark.icon" :class="`w-4 h-4 ${bookmark.icon}`" />
            {{ bookmark.name }}
          </div>
          <div class="text-xs flex items-center gap-1 text-text-2">
            <kbd v-for="(char, i) in bookmark.chord.split('')" :key="i"
              class="bg-bg border border-div px-1 py-0.5 rounded text-sm font-semibold">
              {{ char }}
            </kbd>
          </div>
        </div>
      </button>
    </div>
  </div>
</template>
