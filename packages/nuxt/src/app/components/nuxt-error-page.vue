<template>
  <ErrorTemplate v-bind="{ statusCode, statusMessage, description, stack }" />
</template>

<script setup>
import { defineAsyncComponent } from 'vue'

const props = defineProps({
  error: Object
})

// TODO: extract to a separate utility
const stacktrace = (props.error.stack || '')
  .split('\n')
  .splice(1)
  .map((line) => {
    const text = line
      .replace('webpack:/', '')
      .replace('.vue', '.js') // TODO: Support sourcemap
      .trim()
    return {
      text,
      internal: (line.includes('node_modules') && !line.includes('.cache')) ||
        line.includes('internal') ||
        line.includes('new Promise')
    }
  }).map(i => `<span class="stack${i.internal ? ' internal' : ''}">${i.text}</span>`).join('\n')

// Error page props
const statusCode = Number(props.error.statusCode || 500)
const is404 = statusCode === 404

const statusMessage = props.error.statusMessage ?? (is404 ? 'Page Not Found' : 'Internal Server Error')
const description = props.error.message || props.error.toString()
const stack = process.dev && !is404 ? props.error.description || `<pre>${stacktrace}</pre>` : undefined

// TODO: Investigate side-effect issue with imports
const _Error404 = defineAsyncComponent(() => import('@nuxt/ui-templates/templates/error-404.vue'))
const _Error = process.dev
  ? defineAsyncComponent(() => import('@nuxt/ui-templates/templates/error-dev.vue'))
  : defineAsyncComponent(() => import('@nuxt/ui-templates/templates/error-500.vue'))

const ErrorTemplate = is404 ? _Error404 : _Error
</script>
