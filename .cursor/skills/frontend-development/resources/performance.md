# Performance Optimization

Patterns for optimizing Vue/Nuxt component performance, preventing unnecessary re-renders, and avoiding memory leaks.

---

## Memoization Patterns

### computed for Expensive Computations

```vue
<script setup lang="ts">
import { computed } from 'vue'

interface Props {
  items: Item[]
  searchTerm: string
}

const props = defineProps<Props>()

// ❌ AVOID - Runs on every render
const filteredItems = props.items
  .filter(item => item.name.includes(props.searchTerm))
  .sort((a, b) => a.name.localeCompare(b.name))

// ✅ CORRECT - Memoized, only recalculates when dependencies change
const filteredItems = computed(() => {
  return props.items
    .filter(item => item.name.toLowerCase().includes(props.searchTerm.toLowerCase()))
    .sort((a, b) => a.name.localeCompare(b.name))
})
</script>

<template>
  <List :items="filteredItems" />
</template>
```

**When to use computed:**
- Filtering/sorting large arrays
- Complex calculations
- Transforming data structures
- Expensive computations (loops, recursion)
- Derived state from props or reactive refs

**When NOT to use computed:**
- Simple string concatenation
- Basic arithmetic
- Premature optimization (profile first!)
- Values that change on every render anyway

**Reference:** See `areas/form-shared/app/composables/renderer/builders/useCmsDefaultItemBuilder.ts` for computed usage patterns.

---

## Stable Function References

### The Problem

```vue
<script setup lang="ts">
// ❌ AVOID - Creates new function on every render
const handleClick = (id: string) => {
  console.log('Clicked:', id)
}
</script>

<template>
  <!-- Child re-renders every time Parent renders -->
  <!-- because handleClick is a new function reference each time -->
  <Child :on-click="handleClick" />
</template>
```

### The Solution

```vue
<script setup lang="ts">
import { defineComponent, h } from 'vue'

// ✅ CORRECT - Define function outside setup (stable reference)
function handleClick(id: string) {
  console.log('Clicked:', id)
}

// ✅ OR use arrow function at module level
const handleClick = (id: string) => {
  console.log('Clicked:', id)
}
</script>

<template>
  <!-- Child only re-renders when props actually change -->
  <Child :on-click="handleClick" />
</template>
```

**When to use stable function references:**
- Functions passed as props to children
- Functions used in watch/watchEffect dependencies
- Event handlers in lists
- Functions passed to memoized components

**When NOT to worry:**
- Inline handlers: `@click="() => doSomething()"` (Vue handles these efficiently)
- Event handlers not passed to children

---

## Component Memoization

### defineComponent with markRaw

For dynamically created components, use `markRaw` to prevent Vue from making them reactive:

```typescript
import { defineComponent, markRaw, h } from 'vue'

// Component cache for memoization
const componentCache = new Map()

function getItemComponent(itemId: string, itemComponent: any) {
  const cacheKey = `${itemId}_${itemComponent?.name || 'unknown'}`
  if (!componentCache.has(cacheKey)) {
    // ✅ CORRECT - markRaw prevents reactivity overhead
    const component = markRaw(defineComponent({
      name: `Item_${itemId}`,
      props: ['itemProps', 'itemOn', 'itemSlots'],
      setup(componentProps) {
        return () => h(
          itemComponent,
          {
            ...componentProps.itemProps,
            ...componentProps.itemOn,
          },
          componentProps.itemSlots,
        )
      },
    }))
    componentCache.set(cacheKey, component)
  }
  return componentCache.get(cacheKey)
}
```

**Reference:** See `areas/form-shared/app/composables/renderer/custom/useCustomExperienceItemBuilder.ts` for component memoization patterns.

### shallowRef for VNode Caching

```typescript
import { shallowRef, watch, h } from 'vue'

// ✅ CORRECT - shallowRef for VNodes (prevents deep reactivity)
const vNode = shallowRef(null)

const updateVNode = () => {
  const component = getTaskComponent(currentTaskId, currentTaskType)
  vNode.value = h(component, {
    key: currentKey,
    taskProps: props,
    taskOn: on,
    taskSlots: scopedSlots,
  })
}

watch([() => tasks.value[idx]?.task_type, () => tasks.value[idx]?.id], updateVNode, { immediate: true })
```

**Reference:** See `areas/form-shared/app/composables/renderer/base/usePagesBuilder.ts` for shallowRef usage.

**When to use markRaw/shallowRef:**
- Dynamically created components
- Large component trees
- VNode caching
- Component definitions that don't need reactivity

**When NOT to use:**
- Regular component definitions
- Props that need reactivity
- Premature optimization

---

## Debounced Search

### Using useDebounceFn (VueUse)

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useDebounceFn } from '@vueuse/core'
import { useAsyncData } from '#app'

const searchTerm = ref('')

// Debounce the search function
const debouncedSearch = useDebounceFn(async (term: string) => {
  if (term.length > 0) {
    await refreshSearch()
  }
}, 300)

// Watch search term and trigger debounced search
watch(searchTerm, (newTerm) => {
  debouncedSearch(newTerm)
})

const {
  data: searchResults,
  refresh: refreshSearch,
} = useAsyncData(
  'search',
  () => $http.$get('/api/search', { params: { q: searchTerm.value } }),
  {
    server: false,
    immediate: false,
  }
)
</script>

<template>
  <input
    v-model="searchTerm"
    placeholder="Search..."
  />
</template>
```

### Using useListData with Built-in Debouncing

```vue
<script setup lang="ts">
import { useListData } from '#common/composables'

// ✅ CORRECT - useListData handles debouncing automatically
const {
  listData: files,
  listQuery,
  listLoading,
} = useListData('file-list', {
  query: ['page', 'per_page', 'keyword'],
  defaultLimit: 20,
  server: false,
  async onFetch(query) {
    const response = await api.list({
      keyword: query.keyword,
      page: query.page,
      per_page: query.per_page,
    })
    return response.files
  },
})
</script>

<template>
  <input v-model="listQuery.keyword" placeholder="Search..." />
</template>
```

**Reference:** See `areas/common/app/composables/useListData.ts` for list data patterns.

**Optimal Debounce Timing:**
- **300-500ms**: Search/filtering
- **1000ms**: Auto-save
- **100-200ms**: Real-time validation

---

## Memory Leak Prevention

### Cleanup Timeouts/Intervals

```vue
<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const count = ref(0)

let intervalId: ReturnType<typeof setInterval> | null = null

onMounted(() => {
  // ✅ CORRECT - Cleanup interval
  intervalId = setInterval(() => {
    count.value++
  }, 1000)
})

onUnmounted(() => {
  // ✅ CORRECT - Cleanup on unmount
  if (intervalId) {
    clearInterval(intervalId)
  }
})
</script>
```

### Cleanup Event Listeners

```vue
<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue'

function handleResize() {
  console.log('Resized')
}

onMounted(() => {
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  // ✅ CORRECT - Cleanup event listener
  window.removeEventListener('resize', handleResize)
})
</script>
```

### Cleanup watch/watchEffect

```vue
<script setup lang="ts">
import { watch, watchEffect } from 'vue'

// ✅ CORRECT - watch automatically cleans up
const stopWatcher = watch(source, (newVal) => {
  console.log('Changed:', newVal)
})

// Cleanup manually if needed
onUnmounted(() => {
  stopWatcher()
})

// ✅ CORRECT - watchEffect automatically cleans up
const stopEffect = watchEffect(() => {
  // Reactive code
  console.log('Effect running')
})

// Cleanup manually if needed
onUnmounted(() => {
  stopEffect()
})
</script>
```

### Abort Controllers for Fetch

```vue
<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue'

let abortController: AbortController | null = null

onMounted(async () => {
  abortController = new AbortController()

  try {
    const response = await $http.$get('/api/data', {
      signal: abortController.signal,
    })
    // Handle response
  } catch (error: any) {
    if (error.name === 'AbortError') {
      console.log('Fetch aborted')
    }
  }
})

onUnmounted(() => {
  // ✅ CORRECT - Abort fetch on unmount
  if (abortController) {
    abortController.abort()
  }
})
</script>
```

**Note**: With `useAsyncData` and `useListData`, request cancellation is handled automatically when components unmount.

**Reference:** See `areas/common/app/composables/useListData.ts` for cleanup patterns.

---

## Form Performance

### Watch Specific Fields (Not All)

```vue
<script setup lang="ts">
import { ref, watch } from 'vue'

const formData = ref({
  username: '',
  email: '',
  password: '',
})

// ❌ AVOID - Watches entire object, re-renders on any change
watch(formData, (newVal) => {
  console.log('Form changed:', newVal)
})

// ✅ CORRECT - Watch only what you need
watch(() => formData.value.username, (newUsername) => {
  console.log('Username changed:', newUsername)
})

watch(() => formData.value.email, (newEmail) => {
  console.log('Email changed:', newEmail)
})

// ✅ OR watch multiple specific fields
watch(
  [() => formData.value.username, () => formData.value.email],
  ([newUsername, newEmail]) => {
    console.log('Username or email changed')
  }
)
</script>
```

### Using useForm with Selective Watching

```vue
<script setup lang="ts">
import { useForm } from '#common/composables'

const { formData, watchField } = useForm(templateRef, schema, initData)

// ✅ CORRECT - Watch specific fields only
const username = computed(() => formData.value.username)
const email = computed(() => formData.value.email)

// Only re-renders when username/email change
</script>

<template>
  <input v-model="formData.username" />
  <input v-model="formData.email" />
  <input v-model="formData.password" />
  
  <!-- Only re-renders when username/email change -->
  <p>Username: {{ username }}, Email: {{ email }}</p>
</template>
```

**Reference:** See `areas/common/app/composables/useForm.ts` for form patterns.

---

## List Rendering Optimization

### Key Prop Usage

```vue
<template>
  <!-- ✅ CORRECT - Stable unique keys -->
  <ListItem
    v-for="item in items"
    :key="item.id"
    :item="item"
  />

  <!-- ❌ AVOID - Index as key (unstable if list changes) -->
  <ListItem
    v-for="(item, index) in items"
    :key="index"
    :item="item"
  />
</template>
```

### Memoized List Items with Component Caching

```vue
<script setup lang="ts">
import { defineComponent, markRaw, h, watchEffect } from 'vue'

interface Props {
  items: Item[]
}

const props = defineProps<Props>()

// Component cache for memoization
const componentCache = new Map()

function getItemComponent(itemId: string, itemComponent: any) {
  const cacheKey = `${itemId}_${itemComponent?.name || 'unknown'}`
  if (!componentCache.has(cacheKey)) {
    const component = markRaw(defineComponent({
      name: `Item_${itemId}`,
      props: ['item', 'onAction'],
      setup(componentProps) {
        return () => h(
          'div',
          {
            onClick: () => componentProps.onAction(componentProps.item.id),
          },
          componentProps.item.name
        )
      },
    }))
    componentCache.set(cacheKey, component)
  }
  return componentCache.get(cacheKey)
}

const itemsVNode = ref<any[]>([])

watchEffect(() => {
  itemsVNode.value = props.items.map((item) => {
    const itemComponent = getItemComponent(item.id, ItemComponent)
    return h(itemComponent, {
      key: item.id,
      item,
      onAction: handleAction,
    })
  })
})

function handleAction(id: string) {
  console.log('Action:', id)
}
</script>

<template>
  <component
    v-for="vnode in itemsVNode"
    :is="vnode"
  />
</template>
```

**Reference:** See `areas/form-shared/app/composables/renderer/custom/useCustomExperienceItemBuilder.ts` for list rendering patterns.

---

## Preventing Component Re-initialization

### The Problem

```vue
<script setup lang="ts">
// ❌ AVOID - Component recreated on every render
const ChildComponent = defineComponent({
  setup() {
    return () => h('div', 'Child')
  },
})
</script>

<template>
  <!-- Unmounts and remounts every render -->
  <ChildComponent />
</template>
```

### The Solution

```vue
<script setup lang="ts">
import { defineComponent, markRaw } from 'vue'

// ✅ CORRECT - Define outside setup (stable component)
const ChildComponent = markRaw(defineComponent({
  name: 'ChildComponent',
  setup() {
    return () => h('div', 'Child')
  },
}))
</script>

<template>
  <!-- Stable component reference -->
  <ChildComponent />
</template>
```

### Dynamic Components with Caching

```vue
<script setup lang="ts">
import { computed, markRaw, defineComponent } from 'vue'

interface Props {
  config: Config
}

const props = defineProps<Props>()

// ✅ CORRECT - Cache dynamic components
const componentCache = new Map()

const DynamicComponent = computed(() => {
  const cacheKey = props.config.type
  if (!componentCache.has(cacheKey)) {
    const component = markRaw(defineComponent({
      name: `Dynamic_${cacheKey}`,
      setup() {
        return () => h('div', props.config.title)
      },
    }))
    componentCache.set(cacheKey, component)
  }
  return componentCache.get(cacheKey)
})
</script>

<template>
  <component :is="DynamicComponent" />
</template>
```

**Reference:** See `areas/form-shared/app/composables/renderer/base/usePagesBuilder.ts` for dynamic component patterns.

---

## Lazy Loading Heavy Dependencies

### Code Splitting with Dynamic Imports

```vue
<script setup lang="ts">
// ❌ AVOID - Import heavy libraries at top level
import jsPDF from 'jspdf'  // Large library loaded immediately
import * as XLSX from 'xlsx'  // Large library loaded immediately

// ✅ CORRECT - Dynamic import when needed
async function handleExportPDF() {
  const { jsPDF } = await import('jspdf')
  const doc = new jsPDF()
  // Use it
}

async function handleExportExcel() {
  const XLSX = await import('xlsx')
  // Use it
}
</script>
```

### Lazy Loading Components

```vue
<script setup lang="ts">
import { defineAsyncComponent } from 'vue'

// ✅ CORRECT - Lazy load heavy components
const HeavyDataGrid = defineAsyncComponent(() =>
  import('./components/HeavyDataGrid.vue')
)

const ChartComponent = defineAsyncComponent({
  loader: () => import('./components/ChartComponent.vue'),
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorComponent,
  delay: 200,
  timeout: 3000,
})
</script>

<template>
  <Suspense>
    <template #default>
      <HeavyDataGrid :data="data" />
    </template>
    <template #fallback>
      <LoadingSpinner />
    </template>
  </Suspense>
</template>
```

### Nuxt Auto-Imports with Lazy Loading

```vue
<script setup lang="ts">
// Nuxt automatically code-splits components
// Use <LazyComponentName> prefix for lazy loading
</script>

<template>
  <!-- Automatically lazy loaded -->
  <LazyHeavyComponent v-if="showHeavy" />
</template>
```

---

## watch vs watchEffect

### When to Use watch

```vue
<script setup lang="ts">
import { watch, ref } from 'vue'

const count = ref(0)
const doubled = ref(0)

// ✅ CORRECT - Use watch when you need specific dependencies
watch(count, (newCount) => {
  doubled.value = newCount * 2
})

// ✅ CORRECT - Watch multiple sources
watch([count, otherRef], ([newCount, newOther]) => {
  // React to changes
}, { immediate: true, deep: true })
</script>
```

### When to Use watchEffect

```vue
<script setup lang="ts">
import { watchEffect, ref } from 'vue'

const items = ref<Item[]>([])
const searchTerm = ref('')

// ✅ CORRECT - Use watchEffect when dependencies are implicit
watchEffect(() => {
  // Automatically tracks items and searchTerm
  const filtered = items.value.filter(item =>
    item.name.includes(searchTerm.value)
  )
  // Do something with filtered
})
</script>
```

**When to use watch:**
- You need explicit control over dependencies
- You need to watch specific values
- You need different behavior for old vs new values

**When to use watchEffect:**
- Dependencies are implicit and auto-tracked
- You want reactive code that runs immediately
- Simpler syntax for reactive side effects

**Reference:** See `areas/form-shared/app/composables/renderer/custom/useCustomExperienceItemBuilder.ts` for watchEffect patterns.

---

## Reactive State Optimization

### shallowRef for Large Objects

```vue
<script setup lang="ts">
import { shallowRef, ref } from 'vue'

// ❌ AVOID - Deep reactivity for large objects
const largeData = ref({
  items: Array(10000).fill({ /* ... */ }),
  metadata: { /* ... */ },
})

// ✅ CORRECT - shallowRef for large objects (only top-level reactivity)
const largeData = shallowRef({
  items: Array(10000).fill({ /* ... */ }),
  metadata: { /* ... */ },
})
</script>
```

### markRaw for Non-Reactive Data

```vue
<script setup lang="ts">
import { markRaw } from 'vue'

// ✅ CORRECT - markRaw for third-party instances
const chartInstance = markRaw(new Chart(canvas, config))

// ✅ CORRECT - markRaw for component definitions
const ComponentDefinition = markRaw(defineComponent({ /* ... */ }))
</script>
```

---

## Summary

**Performance Checklist:**
- ✅ `computed` for expensive computations (filter, sort, map)
- ✅ Stable function references (define outside setup)
- ✅ `markRaw` for component definitions and third-party instances
- ✅ `shallowRef` for large objects and VNodes
- ✅ Debounce search/filter (300-500ms) with `useDebounceFn` or `useListData`
- ✅ Cleanup timeouts/intervals in `onUnmounted`
- ✅ Cleanup event listeners in `onUnmounted`
- ✅ Watch specific form fields (not entire objects)
- ✅ Stable keys in lists (use `item.id`, not index)
- ✅ Lazy load heavy libraries with dynamic imports
- ✅ Code splitting with `defineAsyncComponent` or `<LazyComponent>`
- ✅ Use `watch` for explicit dependencies, `watchEffect` for implicit

**See Also:**
- [component-patterns.md](component-patterns.md) - Component structure and patterns
- [data-fetching.md](data-fetching.md) - `useAsyncData` and `useListData` optimization
- [complete-examples.md](complete-examples.md) - Performance patterns in context
- `areas/form-shared/app/composables/renderer/` - Component memoization examples
- `areas/common/app/composables/useListData.ts` - List data patterns
