<script setup>
import { ref } from 'vue'

const openExpression = ref('identity')

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
</script>

<template>
  <main class="page" aria-labelledby="title">
    <header class="header">
      <p class="label">portfolio.lisp</p>
      <h1 id="title">pwnjp</h1>
    </header>

    <section class="repl" aria-label="LISP DSL portfolio">
      <article
        v-for="expression in expressions"
        :key="expression.id"
        class="expression"
        :class="{
          'is-open': openExpression === expression.id,
          'is-evaluable': expression.result,
        }"
      >
        <button
          type="button"
          class="form"
          :disabled="!expression.result"
          :aria-expanded="expression.result ? openExpression === expression.id : undefined"
          @click="toggleExpression(expression)"
        >
          {{ expression.form }}
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
  </main>
</template>
