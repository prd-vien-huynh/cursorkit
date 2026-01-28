# TypeScript Standards

TypeScript best practices for type safety and maintainability in Vue/Nuxt frontend code.

---

## Strict Mode

### Configuration

TypeScript strict mode is **enabled** in the project:

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

**This means:**
- No implicit `any` types
- Null/undefined must be handled explicitly
- Type safety enforced

---

## No `any` Type

### The Rule

```typescript
// ❌ NEVER use any
function handleData(data: any) {
  return data.something
}

// ✅ Use specific types
interface MyData {
  something: string
}

function handleData(data: MyData) {
  return data.something
}

// ✅ Or use unknown for truly unknown data
function handleUnknown(data: unknown) {
  if (typeof data === 'object' && data !== null && 'something' in data) {
    return (data as MyData).something
  }
}
```

**If you truly don't know the type:**
- Use `unknown` (forces type checking)
- Use type guards to narrow
- Document why type is unknown

---

## Explicit Return Types

### Function Return Types

```typescript
// ✅ CORRECT - Explicit return type
async function getUser(id: number): Promise<User> {
  return await $http.$get(`/users/${id}`)
}

function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0)
}

// ❌ AVOID - Implicit return type (less clear)
async function getUser(id: number) {
  return await $http.$get(`/users/${id}`)
}
```

### Composable Return Types

```typescript
// ✅ CORRECT - Explicit return type interface
interface UseMyDataReturn {
  data: Ref<Data | null>
  isLoading: Ref<boolean>
  error: Ref<string | null>
  fetch: () => Promise<void>
}

export function useMyData(id: Ref<number>): UseMyDataReturn {
  const data = ref<Data | null>(null)
  const isLoading = ref(false)
  const error = ref<string | null>(null)

  async function fetch() {
    isLoading.value = true
    try {
      data.value = await $http.$get(`/api/data/${id.value}`)
    } catch (err) {
      error.value = err.message
    } finally {
      isLoading.value = false
    }
  }

  return {
    data,
    isLoading,
    error,
    fetch,
  }
}
```

**Reference:** See `areas/analytics/app/composables/usePolling.ts` for composable return type patterns.

---

## Type Imports

### Use 'type' Keyword

```typescript
// ✅ CORRECT - Explicitly mark as type import
import type { User } from '#common/types/user'
import type { Post } from '#common/types/post'
import type { Ref, ComputedRef } from 'vue'

// ❌ AVOID - Mixed value and type imports (when possible)
import { User } from '#common/types/user'  // Unclear if type or value
```

**Benefits:**
- Clearly separates types from values
- Better tree-shaking
- Prevents circular dependencies
- TypeScript compiler optimization

**When to use `import type`:**
- Interfaces and types
- Type-only utility types
- Type parameters

**When NOT to use `import type`:**
- Classes (can be used as both type and value)
- Enums (can be used as both type and value)
- Values that are actually used at runtime

---

## Component Props

### defineProps with Interface

```vue
<script setup lang="ts">
/**
 * Props for MyComponent
 */
interface MyComponentProps {
  /** The user ID to display */
  userId: number

  /** Optional callback when action completes */
  onComplete?: () => void

  /** Display mode for the component */
  mode?: 'view' | 'edit'

  /** Additional CSS classes */
  class?: string
}

const props = withDefaults(
  defineProps<MyComponentProps>(),
  {
    mode: 'view',  // Default value
  }
)
</script>
```

**Key Points:**
- Separate interface for props
- JSDoc comments for each prop
- Optional props use `?`
- Use `withDefaults` for default values

**Reference:** See `areas/form-shared/app/components/cms-document-dialog/fillable-pdf/edit-document-body-wrapper.vue` for props interface patterns.

### Inline Props (Simple Cases)

```vue
<script setup lang="ts">
// ✅ OK - For simple props without defaults
const props = defineProps<{
  userId: number
  title: string
  onComplete?: () => void
}>()
</script>
```

### Props with Defaults

```vue
<script setup lang="ts">
interface Props {
  value?: number
  form: any
  options?: Record<string, any>
  audienceOptionList?: any[]
}

// ✅ CORRECT - Use withDefaults for function defaults
const props = withDefaults(defineProps<Props>(), {
  value: 0,
  options: () => ({}),  // Function for object/array defaults
  audienceOptionList: () => [],
})
</script>
```

**Reference:** See `areas/form-shared/app/components/cms-document-dialog/fillable-pdf/edit-document-body-wrapper.vue` for withDefaults usage.

---

## Component Emits

### defineEmits with Interface

```vue
<script setup lang="ts">
interface MyComponentEmits {
  (e: 'update', value: string): void
  (e: 'delete', id: number): void
  (e: 'close'): void
  (e: 'update:modelValue', value: string): void  // v-model support
}

const emit = defineEmits<MyComponentEmits>()

function handleClick() {
  emit('update', 'new value')
  emit('delete', 123)
}
</script>
```

### Inline Emits (Simple Cases)

```vue
<script setup lang="ts">
// ✅ OK - For simple emits
const emit = defineEmits<{
  (e: 'update', value: string): void
  (e: 'close'): void
}>()
</script>
```

**Reference:** See `areas/campaign-shared/app/components/message-composer/index.vue` for emit patterns.

### Complex Emit Types

```vue
<script setup lang="ts">
// ✅ CORRECT - Complex emit with payload types
interface AddressEmits {
  (e: 'handle-select-place', place: any): void
  (e: 'handle-switch-manual', isManual: boolean): void
  (
    e: 'get-predictions',
    payload: {
      params: {
        input: string
        country?: string
      }
      resolve: (result: any) => void
    }
  ): void
}

const emit = defineEmits<AddressEmits>()
</script>
```

**Reference:** See `areas/form-shared/app/components/guide-flow/guide-flow-field/guide-flow-search-address-autocomplete.vue` for complex emit patterns.

---

## defineModel (v-model)

### Basic Usage

```vue
<script setup lang="ts">
// ✅ CORRECT - Typed v-model
const modelValue = defineModel<string>({
  default: '',
})

// With options
const count = defineModel<number>('count', {
  default: 0,
  required: true,
})
</script>

<template>
  <input v-model="modelValue" />
</template>
```

### Multiple v-models

```vue
<script setup lang="ts">
// ✅ CORRECT - Multiple v-models
const composeChannel = defineModel<CommunicationChannel>('compose-channel', {
  default: CommunicationMethod.SMS,
})

const introMessage = defineModel<string>('intro_message_sms', {
  default: '',
})
</script>
```

**Reference:** See `areas/campaign-shared/app/components/message-composer/index.vue` for defineModel patterns.

---

## Utility Types

### Partial<T>

```typescript
// Make all properties optional
type UserUpdate = Partial<User>

function updateUser(id: number, updates: Partial<User>) {
  // updates can have any subset of User properties
}
```

### Pick<T, K>

```typescript
// Select specific properties
type UserPreview = Pick<User, 'id' | 'name' | 'email'>

const preview: UserPreview = {
  id: 1,
  name: 'John',
  email: 'john@example.com',
  // Other User properties not allowed
}
```

### Omit<T, K>

```typescript
// Exclude specific properties
type UserWithoutPassword = Omit<User, 'password' | 'passwordHash'>

const publicUser: UserWithoutPassword = {
  id: 1,
  name: 'John',
  email: 'john@example.com',
  // password and passwordHash not allowed
}
```

### Required<T>

```typescript
// Make all properties required
type RequiredConfig = Required<Config>  // All optional props become required
```

### Record<K, V>

```typescript
// Type-safe object/map
const userMap: Record<string, User> = {
  'user1': { id: 1, name: 'John' },
  'user2': { id: 2, name: 'Jane' },
}

// For component refs
const fieldRefs = reactive<Record<string, any>>({
  address: null,
  city: null,
  state: null,
})
```

### Vue-Specific Utility Types

```typescript
import type { Ref, ComputedRef, MaybeRef } from 'vue'

// Ref types
const count: Ref<number> = ref(0)

// ComputedRef types
const doubled: ComputedRef<number> = computed(() => count.value * 2)

// MaybeRef (can be ref or value)
function useUnref<T>(value: MaybeRef<T>): T {
  return isRef(value) ? value.value : value
}
```

---

## Type Guards

### Basic Type Guards

```typescript
function isUser(data: unknown): data is User {
  return (
    typeof data === 'object' &&
    data !== null &&
    'id' in data &&
    'name' in data
  )
}

// Usage
if (isUser(response)) {
  console.log(response.name)  // TypeScript knows it's User
}
```

### Discriminated Unions

```typescript
type LoadingState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: Data }
  | { status: 'error'; error: Error }

function Component({ state }: { state: LoadingState }) {
  // TypeScript narrows type based on status
  if (state.status === 'success') {
    return state.data  // data available here
  }

  if (state.status === 'error') {
    return state.error  // error available here
  }

  return null
}
```

---

## Generic Types

### Generic Functions

```typescript
function getById<T>(items: T[], id: number): T | undefined {
  return items.find(item => (item as any).id === id)
}

// Usage with type inference
const users: User[] = [...]
const user = getById(users, 123)  // Type: User | undefined
```

### Generic Composables

```typescript
interface UseListDataOptions<T, K extends Readonly<TupleOf<3, string>>> {
  server?: boolean
  immediate?: boolean
  onFetch: (query: Query<K>) => Promise<T[] | undefined | null>
  // ... other options
}

export function useListData<
  T,
  K extends Readonly<TupleOf<3, string>> = ['page', 'limit', 'keyword'],
>(
  key: string,
  options: UseListDataOptions<T, K>,
) {
  // Implementation
}
```

**Reference:** See `areas/common/app/composables/useListData.ts` for generic composable patterns.

### Generic Components

```vue
<script setup lang="ts" generic="T">
interface ListProps<T> {
  items: T[]
  renderItem: (item: T) => any
}

const props = defineProps<ListProps<T>>()
</script>

<template>
  <div v-for="item in props.items" :key="item.id">
    <component :is="props.renderItem(item)" />
  </div>
</template>
```

---

## Type Assertions (Use Sparingly)

### When to Use

```typescript
// ✅ OK - When you know more than TypeScript
const element = document.getElementById('my-element') as HTMLInputElement
const value = element.value

// ✅ OK - API response that you've validated
const response = await $http.$get('/api/data')
const user = response.data as User  // You know the shape

// ✅ OK - Vue component instance
const vm = getCurrentInstance()?.proxy as ComponentPublicInstance
```

### When NOT to Use

```typescript
// ❌ AVOID - Circumventing type safety
const data = getData() as any  // WRONG - defeats TypeScript

// ❌ AVOID - Unsafe assertion
const value = unknownValue as string  // Might not actually be string
```

---

## Null/Undefined Handling

### Optional Chaining

```typescript
// ✅ CORRECT
const name = user?.profile?.name

// Equivalent to:
const name = user && user.profile && user.profile.name
```

### Nullish Coalescing

```typescript
// ✅ CORRECT
const displayName = user?.name ?? 'Anonymous'

// Only uses default if null or undefined
// (Different from || which triggers on '', 0, false)
```

### Non-Null Assertion (Use Carefully)

```typescript
// ✅ OK - When you're certain value exists
const data = useState<Data>('data')!
if (data.value) {
  // Use data
}

// ⚠️ CAREFUL - Only use when you KNOW it's not null
// Better to check explicitly:
const data = useState<Data>('data')
if (data.value) {
  // Use data.value
}
```

### Vue-Specific Null Handling

```vue
<script setup lang="ts">
// ✅ CORRECT - Handle refs that might be null
const elementRef = ref<HTMLInputElement | null>(null)

onMounted(() => {
  if (elementRef.value) {
    elementRef.value.focus()
  }
})

// ✅ CORRECT - Optional chaining with refs
const value = computed(() => props.data?.items?.[0]?.name ?? 'Default')
</script>
```

---

## Composable Patterns

### Composable Return Type Interface

```typescript
// ✅ CORRECT - Explicit return type
interface UsePollingReturn<T> {
  data: Ref<T | null>
  loading: Ref<boolean>
  error: Ref<string | null>
  status: Ref<QueryStatus | null>
  startPolling: (params: ReportDataParams) => Promise<void>
  stopPolling: () => void
  reset: () => void
}

export function usePolling<T extends { status?: QueryStatus }>(
  fetchFn: (params: ReportDataParams) => Promise<T>,
  options: UsePollingOptions<T> = {},
): UsePollingReturn<T> {
  // Implementation
  return {
    data,
    loading,
    error,
    status,
    startPolling,
    stopPolling,
    reset,
  }
}
```

**Reference:** See `areas/analytics/app/composables/usePolling.ts` for composable return type patterns.

### Composable Options Interface

```typescript
// ✅ CORRECT - Options interface
interface UseListDataOptions<T, K extends Readonly<TupleOf<3, string>>> {
  server?: boolean
  immediate?: boolean
  lazy?: boolean
  default?: () => T[]
  onFetch: (query: Query<K>) => Promise<T[] | undefined | null>
  postFetch?: (data: T[], options: { loadmore: boolean }) => T[]
}

export function useListData<T, K>(
  key: string,
  options: UseListDataOptions<T, K>,
) {
  // Implementation
}
```

**Reference:** See `areas/common/app/composables/useListData.ts` for composable options patterns.

---

## defineOptions

### Component Name

```vue
<script setup lang="ts">
// ✅ CORRECT - Define component name for debugging
defineOptions({
  name: 'MyComponent',
})

const props = defineProps<{
  // props
}>()
</script>
```

**Reference:** See `areas/cem-shared/app/components/genai/toolbar.vue` for defineOptions usage.

---

## Reactive Types

### ref Types

```typescript
// ✅ CORRECT - Explicit ref types
const count = ref<number>(0)
const user = ref<User | null>(null)
const items = ref<Item[]>([])

// ✅ CORRECT - Typed useState (Nuxt)
const siteData = useState<Site>('site-data', () => null)
```

### computed Types

```typescript
// ✅ CORRECT - Computed with explicit return type
const filteredItems = computed<Item[]>(() => {
  return items.value.filter(item => item.active)
})

// TypeScript infers from return value, but explicit is clearer
```

### watch Types

```typescript
// ✅ CORRECT - Typed watch
watch(
  (): User => user.value,
  (newUser: User, oldUser: User) => {
    // Handle change
  }
)

// ✅ CORRECT - Multiple sources
watch(
  [() => count.value, () => name.value],
  ([newCount, newName], [oldCount, oldName]) => {
    // Handle changes
  }
)
```

---

## API Response Types

### Typed API Calls

```typescript
// ✅ CORRECT - Type API responses
interface UserResponse {
  data: User
}

async function getUser(id: string): Promise<UserResponse> {
  return await $http.$get<UserResponse>(`/users/${id}`)
}

// Usage
const response = await getUser('123')
const user = response.data  // Typed as User
```

### useAsyncData Types

```vue
<script setup lang="ts">
interface SiteDetail {
  site_name: string
  site_id: string
}

const {
  data: siteDetail,
  error: siteError,
  pending: sitePending,
} = useAsyncData<SiteDetail>(
  'site-details',
  () => getSiteDetails(unref(siteId)),
  {
    server: false,
    watch: [siteId],
  }
)
</script>
```

**Reference:** See `areas/common/app/composables/useListData.ts` and data-fetching.md for API typing patterns.

---

## Summary

**TypeScript Checklist:**
- ✅ Strict mode enabled
- ✅ No `any` type (use `unknown` if needed)
- ✅ Explicit return types on functions and composables
- ✅ Use `import type` for type imports
- ✅ JSDoc comments on prop interfaces
- ✅ `defineProps<Props>` with interfaces
- ✅ `defineEmits<Emits>` with interfaces
- ✅ `withDefaults` for prop defaults
- ✅ Utility types (Partial, Pick, Omit, Required, Record)
- ✅ Type guards for narrowing
- ✅ Optional chaining and nullish coalescing
- ✅ Explicit types for refs, computed, watch
- ❌ Avoid type assertions unless necessary

**See Also:**
- [component-patterns.md](component-patterns.md) - Component structure and typing
- [data-fetching.md](data-fetching.md) - API typing patterns
- [performance.md](performance.md) - TypeScript performance considerations
- `areas/common/app/composables/useListData.ts` - Generic composable patterns
- `areas/analytics/app/composables/usePolling.ts` - Composable return types
