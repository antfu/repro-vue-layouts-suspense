<script setup>
import { Transition } from 'vue'
import ContentSync from './Content.vue'

const Content = defineAsyncComponent(() => import('./Content.vue'))

const layout = ref('layoutA')

const layouts = {
  layoutA: () => import('./LayoutA.vue'),
  layoutB: () => import('./LayoutB.vue'),
}

const resolvedLayouts = {

}

function getLayout(name) {
  if (resolvedLayouts[name])
    return resolvedLayouts[name]
  const layout = layouts[name]()
    .then(r => r.default || r)
  resolvedLayouts[name] = layout
  return layout
}

// const Layout = computed(() => layouts[layout.value])

const ENABLE_TRANSITION = false
const TransitionWarpper = ENABLE_TRANSITION
  ? Transition
  : { render() { return this.$slots.default?.() } }

const Layout = defineComponent({
  async setup(_, { slots }) {
    const current = shallowRef()

    const reactive = computed(() => getLayout(layout.value))

    watch(reactive, async (v) => {
      current.value = await v
      console.log('Layout changed')
    })

    current.value = await reactive.value

    return () => h(current.value, null, slots)
  },
})
</script>

<template>
  <Suspense>
    <div>
      <button @click="layout = layout === 'layoutA' ? 'layoutB' : 'layoutA'">
        Toggle
      </button>
      <TransitionWarpper>
        <!-- <Suspense suspensible> -->
        <Layout>
          <!-- <Suspense suspensible> -->
          <ContentSync />
          <!-- </Suspense> -->
        </Layout>
        <!-- </Suspense> -->
      </TransitionWarpper>
    </div>
  </Suspense>
</template>
