<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

const openExpression = ref('identity')
const cursor = ref({ row: 0, column: 0 })

const expressions = [
  {
    id: 'identity',
    form: '(whoami :name "Ryosuke Shimizu")',
    result: '=> (:role "software engineer" :place "Japan")',
  },
  {
    id: 'focus',
    form: '(focus :on ("web security" "backend systems" "developer tools"))',
    result: '=> ("threat-aware interfaces" "small reliable APIs" "useful automation")',
  },
  {
    id: 'stack',
    form: '(stack :uses ("Python" "Golang" "Terraform"))',
    result: '=> (:language ("Python" "Golang") :infrastructure "Terraform")',
  },
  {
    id: 'skill-certifications',
    form: '(skill :certifications)',
    result:
      '=> (progn\n  (certification :name "普通自動車運転免許（第一種）" :acquired "2011-08")\n  (certification :name "情報セキュリティスペシャリスト" :acquired "2014-06")\n  (certification :name "応用情報技術者" :acquired "2016-12")\n  (certification :name "SANS GWAPT" :acquired "2017-12")\n  (certification :name "基本情報技術者試験" :acquired "2018-11"))',
  },
  {
    id: 'pr-platform',
    form: '(self-pr :theme "cloud-data-platform" :clients "enterprise")',
    result:
      '=> "大手企業向けに、クラウド環境でデータ処理基盤の構築・運用・改善に従事。大規模な社内業務システムの安定稼働と効率化に貢献しました。"',
  },
  {
    id: 'pr-implementation',
    form: '(self-pr :strength ("Python" "Go" "AWS" "Terraform"))',
    result:
      '=> "AWS環境下でのデータ整形・加工システムの開発、PythonやGoを用いたバックエンド実装、Terraformによるデータパイプライン構築に強みがあります。"',
  },
  {
    id: 'pr-automation',
    form: '(self-pr :automation ("ai-ops" "incident-response" "data-processing"))',
    result:
      '=> "AIを活用したインシデント対応の自動化システム導入経験があり、大規模データの効率的な処理と安定稼働に貢献してきました。"',
  },
  {
    id: 'contact',
    form: '(contact :github "https://github.com/fvi-att")',
    result: '=> (:github "https://github.com/fvi-att" :status "open")',
    links: [
      {
        label: 'https://github.com/fvi-att',
        href: 'https://github.com/fvi-att',
      },
    ],
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
          <div
            v-if="expression.result && openExpression === expression.id"
            class="result"
          >
            <p>{{ expression.result }}</p>
            <a
              v-for="link in expression.links"
              :key="link.href"
              class="result-link"
              :href="link.href"
              target="_blank"
              rel="noreferrer"
            >
              {{ link.label }}
            </a>
          </div>
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
