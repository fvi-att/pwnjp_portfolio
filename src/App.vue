<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

const openExpression = ref('identity')
const cursor = ref({ row: 0, column: 0 })

const expressions = [
  {
    id: 'identity',
    form: '(whoami :name "pwnjp")',
    result: '=> (:role "software engineer" :place "Japan")',
  },
  {
    id: 'focus',
    form: '(focus :on ("web security" "backend systems" "developer tools"))',
    result: '=> ("threat-aware interfaces" "small reliable APIs" "useful automation")',
  },
  {
    id: 'stack',
    form: '(stack :uses ("Vue" "Vite" "Go" "TypeScript" "Python" "Linux"))',
    result: '=> (:frontend "Vue/Vite" :systems "Go/Linux" :scripts "Python")',
  },
  {
    id: 'portfolio',
    form: '(project :name "portfolio.dsl" :state :active)',
    result: '=> (:idea "portfolio as executable structure")',
  },
  {
    id: 'ctf',
    form: '(project :name "ctf-lab" :state :research)',
    result: '=> (:contains "writeups" "small labs" "tooling notes")',
  },
  {
    id: 'contact',
    form: '(contact :github "github.com/pwnjp" :mail "hello@pwnjp.dev")',
    result: '=> (:status "open")',
  },
  {
    id: 'comment',
    form: ';; click expressions to eval',
  },
]

const toggleExpression = (expression) => {
  if (!expression.result) return
  openExpression.value = openExpression.value === expression.id ? '' : expression.id
}

const activeExpression = computed(() => expressions[cursor.value.row])

const clampColumn = (row, column) => {
  const maxColumn = expressions[row].form.length - 1
  return Math.max(0, Math.min(column, maxColumn))
}

const moveCursor = (rowDelta, columnDelta = 0) => {
  const nextRow = Math.max(
    0,
    Math.min(cursor.value.row + rowDelta, expressions.length - 1),
  )
  cursor.value = {
    row: nextRow,
    column: clampColumn(nextRow, cursor.value.column + columnDelta),
  }
}

const setCursor = (row, column = cursor.value.column) => {
  cursor.value = {
    row,
    column: clampColumn(row, column),
  }
}

const setCursorToLineEnd = (row) => {
  setCursor(row, expressions[row].form.length - 1)
}

const evalActiveExpression = () => {
  if (activeExpression.value?.result) {
    toggleExpression(activeExpression.value)
  }
}

const handleKeydown = (event) => {
  const control = event.ctrlKey
  const key = event.key.toLowerCase()

  if (control && key === 'n') {
    event.preventDefault()
    moveCursor(1)
    return
  }

  if (control && key === 'p') {
    event.preventDefault()
    moveCursor(-1)
    return
  }

  if (control && key === 'f') {
    event.preventDefault()
    moveCursor(0, 1)
    return
  }

  if (control && key === 'b') {
    event.preventDefault()
    moveCursor(0, -1)
    return
  }

  if (control && key === 'a') {
    event.preventDefault()
    setCursor(cursor.value.row, 0)
    return
  }

  if (control && key === 'e') {
    event.preventDefault()
    setCursorToLineEnd(cursor.value.row)
    return
  }

  if (event.key === 'ArrowDown') {
    event.preventDefault()
    moveCursor(1)
    return
  }

  if (event.key === 'ArrowUp') {
    event.preventDefault()
    moveCursor(-1)
    return
  }

  if (event.key === 'ArrowRight') {
    event.preventDefault()
    moveCursor(0, 1)
    return
  }

  if (event.key === 'ArrowLeft') {
    event.preventDefault()
    moveCursor(0, -1)
    return
  }

  if (event.key === 'Enter') {
    event.preventDefault()
    evalActiveExpression()
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeydown)
})

const beforeCursor = (expression, row) =>
  expression.form.slice(0, row === cursor.value.row ? cursor.value.column : 0)

const cursorCharacter = (expression) => expression.form[cursor.value.column] || ' '

const afterCursor = (expression, row) =>
  expression.form.slice(row === cursor.value.row ? cursor.value.column + 1 : 0)

const characterClass = (character) => ({
  paren: character === '(' || character === ')',
})
</script>

<template>
  <main class="page" aria-label="pwnjp portfolio">
    <header class="header">
      <p class="label">portfolio.lisp</p>
    </header>

    <section class="repl" aria-label="LISP DSL portfolio">
      <article
        v-for="(expression, row) in expressions"
        :key="expression.id"
        class="expression"
        :class="{
          'is-open': openExpression === expression.id,
          'is-evaluable': expression.result,
          'is-current': cursor.row === row,
        }"
      >
        <span class="line-number" aria-hidden="true">{{ row + 1 }}</span>
        <button
          type="button"
          class="form"
          :aria-disabled="!expression.result"
          :aria-expanded="expression.result ? openExpression === expression.id : undefined"
          @click="setCursor(row); toggleExpression(expression)"
        >
          <span v-if="cursor.row === row" class="line-text">
            <span
              v-for="(character, index) in beforeCursor(expression, row)"
              :key="`before-${index}`"
              :class="characterClass(character)"
            >{{ character }}</span><span
              class="cursor-char"
              :class="characterClass(cursorCharacter(expression))"
            >{{ cursorCharacter(expression) }}</span><span
              v-for="(character, index) in afterCursor(expression, row)"
              :key="`after-${index}`"
              :class="characterClass(character)"
            >{{ character }}</span>
          </span>
          <span v-else class="line-text">
            <span
              v-for="(character, index) in expression.form"
              :key="index"
              :class="characterClass(character)"
            >{{ character }}</span>
          </span>
        </button>

        <Transition name="eval">
          <p
            v-if="expression.result && openExpression === expression.id"
            class="result"
          >
            {{ expression.result }}
          </p>
        </Transition>
      </article>
    </section>

    <footer class="mode-line">
      <span>portfolio.lisp</span>
      <span>L{{ cursor.row + 1 }} C{{ cursor.column + 1 }}</span>
      <span>Lisp Interaction</span>
    </footer>
  </main>
</template>
