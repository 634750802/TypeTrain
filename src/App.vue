<template>
  <main @keypress.prevent.stop.capture="onKeypress" autofocus tabindex="0" class="focus-within:outline-none min-h-full">
    <header class="bg-white shadow">
      <div class="max-w-7xl mx-auto py-6 px-4 sm:px-6 lg:px-8">
        <h1 class="text-3xl font-bold leading-tight text-gray-900">Type Train</h1>
      </div>
    </header>
    <article class="container p-4 font-mono">
      <template
          v-for="(word, wordIndex) in state.words"
          :key="wordIndex">
        <template v-if="word === '⤶'">
          <span class="text-gray-300">{{ word }}</span>
          <br class="h-2">
        </template>
        <span
            v-else
            class="inline-block mr-1 "
            :class="{
              'text-gray-400': state.cursor.w > wordIndex
            }"
        >
          <template v-if="wordIndex === state.cursor.w || wordIndex === state.cursor.w - 1">
            <span
                class="inline-block"
                :class="{
                  'text-gray-400 animate__animated animate__shakeY':
                    state.cursor.w > wordIndex || state.cursor.c > charIndex,
                }"
                v-for="(char, charIndex) in word"
                :key="charIndex"
            >
              {{ char === ' ' ? '&nbsp;' : char }}
            </span>
          </template>
          <span v-else>{{ word }}</span>
        </span>
      </template>
    </article>
  </main>
</template>
<script>
  import { reactive, defineComponent, computed, onMounted } from 'vue'
  import Chance from 'chance'
  import 'animate.css/animate.css'

  const chance = Chance()

  export default defineComponent({
    setup () {
      const state = reactive({
        words: [],
        cursor: {
          w: 0,
          c: 0
        }
      })

      const currentWord = computed(() => state.words[state.cursor.w])
      const currentChar = computed(() => currentWord?.value[state.cursor.c])

      const set = (text) => {
        state.words = text.split('\n').flatMap(line => {
          const words = line.split(' ').filter(s => s)
          if (words.length > 0) {
            words[words.length - 1] += '\n'
            return words.concat('⤶')
          } else {
            return []
          }
        })
        // console.log('New content', state.words)
      }

      const move = (next) => {
        const { words, cursor } = state

        cursor.c += 1
        if (next) {
          if (cursor.w < words.length && cursor.c >= currentWord.value.length) {
            cursor.c = 0
            cursor.w += 1
          }
        }

        if (cursor.w === words.length) {
          done()
        }
      }

      const checkDone = () => {
        const { words, cursor } = state
        // console.log('check done', cursor.w, cursor.c)
        if (cursor.w === words.length - 1 && cursor.c === currentWord.value.length) {
          done()
        } else if (cursor.w === words.length) {
          done()
        }
      }

      const done = () => {
        const lines = chance.integer({
          max: 12
        })
        let content = ''
        for (let i = 0; i < lines; i++) {
          content += chance.paragraph() + '\n'
        }
        set(content)
        state.cursor.c = 0
        state.cursor.w = 0
      }

      onMounted(() => set('Hello world!\n'))

      return {
        state,
        onKeypress ({ key }) {
          // console.log(key, currentWord.value, currentChar.value)
          if (currentChar.value === key) {
            move(false)
          } else if (key === 'Enter') {
            if (currentChar.value === '\n') {
              move(true)
              move(true)
            } else {
              checkDone()
            }
          } else if (currentChar.value === undefined && key === ' ') {
            move(true)
          }
        }
      }
    }
  })

</script>
