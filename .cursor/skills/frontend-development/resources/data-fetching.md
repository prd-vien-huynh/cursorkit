# Data Fetching Patterns

Modern data fetching in Nuxt 3 using `useAsyncData`, `useListData`, and centralized API services with `$http`.

---

## PRIMARY PATTERN: useAsyncData

### Why useAsyncData?

For **single data fetches** in Vue components, use Nuxt's `useAsyncData`:

**Benefits:**
- Built-in SSR support
- Automatic request deduplication
- Loading state management
- Error handling integration
- Type-safe responses

### Basic Pattern

```vue
<script setup lang="ts">
import { useApiSiteDetails } from '#site-studio/composables'

const { getSiteDetails } = useApiSiteDetails()
const siteId = computed(() => route.params.id as string)

const {
  data: siteDetail,
  error: siteError,
  pending: sitePending,
  refresh: refreshSiteDetail,
} = useAsyncData(
  'site-details',
  () => getSiteDetails(unref(siteId)),
  {
    server: false,
    watch: [siteId],
  }
)

whenError(siteError, () => navigateTo('/site-studio'))
</script>

<template>
  <PageBody v-loading="sitePending">
    <div v-if="siteDetail?.data">
      {{ siteDetail.data.site_name }}
    </div>
  </PageBody>
</template>
```

### useAsyncData Options

```typescript
const { data, error, pending, refresh, execute } = useAsyncData(
  key: string,                    // Unique key for caching
  handler: () => Promise<T>,      // Async function
  options?: {
    server?: boolean              // Run on server (default: true)
    lazy?: boolean                // Don't block navigation (default: false)
    immediate?: boolean          // Execute immediately (default: true)
    watch?: WatchSource[]         // Reactive sources to watch
    default?: () => T            // Default value
    transform?: (data: any) => T // Transform response
    getCachedData?: (key: string) => T | null // Custom cache getter
  }
)
```

**Key Options:**
- `server: false` - Client-only fetch (use for authenticated requests)
- `watch: [ref1, ref2]` - Refetch when watched values change
- `lazy: true` - Don't block page navigation
- `immediate: false` - Don't fetch on mount (call `execute()` manually)

---

## Paginated Lists: useListData

### Why useListData?

For **paginated lists with search**, use the custom `useListData` composable:

**Benefits:**
- Built-in pagination
- Search with debouncing
- Load more functionality
- Server/client state management
- Automatic query parameter handling

### Basic Pattern

```vue
<script setup lang="ts">
import { useApiJobDataPackages } from '#datafeed/composables'

const {
  fetchMoreData: fetchMoreFiles,
  fetchData: fetchFiles,
  listData: files,
  listQuery,
  listLoading,
  canLoadMore,
} = useListData('job-data-packages-file-list', {
  query: ['page', 'per_page', 'keyword'],
  defaultLimit: 20,
  server: false,
  lazy: true,
  immediate: false,
  async onFetch(query) {
    const response = await useApiJobDataPackages().list({
      keyword: query.keyword,
      page: query.page,
      per_page: query.per_page,
    })
    return response.data_feed_files
  },
  onSearch: (query) => {
    // Optional: handle search state
    isSearching.value = Boolean(query.keyword)
  },
})

onMounted(() => {
  fetchFiles()
})
</script>

<template>
  <div>
    <input v-model="listQuery.keyword" placeholder="Search..." />
    <div v-for="file in files" :key="file.uuid">
      {{ file.name }}
    </div>
    <button
      v-if="canLoadMore"
      :disabled="listLoading"
      @click="fetchMoreFiles"
    >
      Load More
    </button>
  </div>
</template>
```

### useListData Options

```typescript
const {
  listData,           // Ref<T[]> - The list items
  listQuery,          // Ref<Query> - Pagination/search query
  listLoading,       // Ref<boolean> - Loading state
  listStatus,        // Ref<'idle' | 'pending' | 'error' | 'success'>
  canLoadMore,       // Computed<boolean> - Can load more
  isFetched,         // Ref<boolean> - Has fetched at least once
  fetchData,         // (opts?) => Promise<T[]>
  fetchMoreData,     // (state?) => Promise<void>
  resetQuery,        // (opts?) => void
  resetData,         // () => void
} = useListData<T, K>(
  key: string,
  options: {
    server?: boolean              // Use useState (default: true)
    immediate?: boolean           // Fetch on mount (default: true)
    lazy?: boolean                // Lazy loading (default: false)
    default?: () => T[]           // Default empty array
    useOffset?: boolean            // Use offset instead of page
    defaultPage?: number          // Default page (default: 1)
    defaultOffset?: number         // Default offset (default: 0)
    defaultLimit?: number          // Default limit (default: 20)
    defaultKeyword?: string        // Default search keyword
    query?: [string, string, string] // Custom query keys [page, limit, keyword]
    watch?: WatchSource[]          // Additional watchers
    onSearch?: (query) => void    // Search callback
    onFetch: (query) => Promise<T[] | null | undefined>
    postFetch?: (data: T[], opts: { loadmore: boolean }) => T[]
    withLoading?: boolean         // Show global loading (default: false)
  }
)
```

**Common Patterns:**
- `server: false` - Client-only lists (authenticated data)
- `immediate: false` - Manual fetch control
- `query: ['page', 'per_page', 'keyword']` - Custom query parameter names
- `useOffset: true` - Use offset-based pagination instead of page numbers

---

## API Service Layer Pattern

### File Structure

Create centralized API service per feature area:

```
areas/
  my-feature/
    app/
      api/
        myFeatureApi.ts    # Service layer
      composables/
        useApiMyFeature.ts # Composable wrapper
```

### Service Pattern

```typescript
// areas/my-feature/app/api/myFeatureApi.ts
import type { MyEntity, CreatePayload, UpdatePayload } from '../types'

const ENDPOINT = '/my-feature/entities'

export default function () {
  /**
   * Fetch a single entity
   */
  async function getEntity(id: string) {
    return await $http.$get<{ data: MyEntity }>(`${ENDPOINT}/${id}`)
  }

  /**
   * Fetch list of entities
   */
  async function list(params: { page?: number, per_page?: number, keyword?: string }) {
    return await $http.$get<{ rows: MyEntity[] }>(ENDPOINT, { params })
  }

  /**
   * Create entity
   */
  async function create(payload: CreatePayload) {
    return await $http.$post<{ data: MyEntity }>(ENDPOINT, { body: payload })
  }

  /**
   * Update entity
   */
  async function update(id: string, payload: UpdatePayload) {
    return await $http.$put<{ data: MyEntity }>(`${ENDPOINT}/${id}`, { body: payload })
  }

  /**
   * Delete entity
   */
  async function deleteEntity(id: string) {
    return await $http.$delete<{ success: boolean }>(`${ENDPOINT}/${id}`)
  }

  return {
    getEntity,
    list,
    create,
    update,
    deleteEntity,
  }
}
```

### Composable Wrapper

```typescript
// areas/my-feature/app/composables/useApiMyFeature.ts
import myFeatureApi from '../api/myFeatureApi'

export function useApiMyFeature() {
  return myFeatureApi()
}
```

**Key Points:**
- Default export function that returns an object with methods
- Use `$http.$get`, `$http.$post`, `$http.$put`, `$http.$delete`
- Type-safe parameters and return types
- Centralized endpoint constants
- JSDoc comments for each method

---

## Route Format Rules (IMPORTANT)

### Correct Format

```typescript
// ✅ CORRECT - Direct service path
await $http.$get('/settings/data-feeds/job-data-packages')
await $http.$post('/site-builder/sites', { body: payload })
await $http.$put('/users/update/456', { body: updates })
await $http.$delete('/settings/data-feeds/job-data-packages/123')

// ❌ WRONG - Do NOT add /api/ prefix
await $http.$get('/api/settings/data-feeds/job-data-packages')  // WRONG!
```

**Microservice Routing:**
- Settings: `/settings/*`
- Site Builder: `/site-builder/*`
- Users: `/users/*`
- Auth: `/auth/*`

**Why:** API routing is handled by proxy configuration, no `/api/` prefix needed.

### $http Methods

```typescript
// GET request
$http.$get<T>(url: string, options?: {
  params?: Record<string, any>
  apiVersion?: string
  signal?: AbortSignal
})

// POST request
$http.$post<T>(url: string, options?: {
  body?: any
  params?: Record<string, any>
  apiVersion?: string
  headers?: Record<string, string>
  responseType?: 'blob' | 'json'
})

// PUT request
$http.$put<T>(url: string, options?: {
  body?: any
  params?: Record<string, any>
  apiVersion?: string
})

// DELETE request
$http.$delete<T>(url: string, options?: {
  params?: Record<string, any>
  apiVersion?: string
})
```

---

## Mutations (POST/PUT/DELETE)

### Basic Mutation Pattern

```vue
<script setup lang="ts">
import { useApiSiteDetails } from '#site-studio/composables'

const { updateSite } = useApiSiteDetails()
const isLoading = ref(false)

async function handleSave() {
  return withCatch(
    async () => {
      isLoading.value = true
      OlLoading()

      const response = await updateSite(siteId.value, formData.value)

      OlNotify.success?.({
        message: $gettext('Site updated successfully'),
      })

      // Refresh data after update
      await refreshSiteDetail()

      return response
    },
    {
      notify: false, // Handle errors manually
      onError: (error) => {
        const errors = error.data?.errors || []
        if (errors.length) {
          setServerErrors(errors)
        } else {
          OlNotify.error?.({
            message: $gettext('Failed to update site'),
          })
        }
      },
      onFinally: () => {
        isLoading.value = false
        OlLoading()?.close()
      },
    }
  )
}
</script>
```

### Error Handling with withCatch

```typescript
import { withCatch } from '#common/utils/error'

withCatch(
  async () => {
    // Your async operation
    return await api.create(data)
  },
  {
    notify?: boolean              // Show error notification (default: false)
    onError?: (error) => void     // Custom error handler
    onFinally?: () => void        // Always runs
  }
)
```

**Key Points:**
- Always wrap mutations in `withCatch`
- Use `notify: false` for custom error handling
- Show loading with `OlLoading()` / `OlLoading()?.close()`
- Show success with `OlNotify.success()`
- Refresh related queries after success

---

## Error Handling

### whenError for useAsyncData

```typescript
import { whenError } from '#common/utils/error'

const { data, error } = useAsyncData('key', fetchFn)

// Automatically watch error and handle it
whenError(error, (err) => {
  // Handle error
  navigateTo('/error-page')
})
```

### withCatch for Manual Operations

```typescript
import { withCatch } from '#common/utils/error'

await withCatch(
  async () => {
    return await api.operation()
  },
  {
    notify: true, // Show error toast automatically
    onError: (error) => {
      // Custom error handling
      console.error('Operation failed:', error)
    },
    onFinally: () => {
      // Cleanup
    },
  }
)
```

### Error Response Structure

```typescript
interface ErrorResponse {
  data?: {
    errors?: Array<{
      field: string | string[]
      code: number
      message: string
    }>
    error?: string
  }
  response?: {
    _data?: any
  }
}
```

---

## Advanced Patterns

### Polling Data

```typescript
import { usePolling } from '#analytics/composables/usePolling'

const {
  data,
  loading,
  error,
  status,
  startPolling,
  stopPolling,
} = usePolling(
  (params) => reportApi.getReportData(params),
  {
    interval: 3000,        // Poll every 3 seconds
    maxAttempts: 180000,  // Max attempts
    autoStart: false,     // Don't start automatically
    onSuccess: (data) => {
      // Handle success
    },
    onError: (error) => {
      // Handle error
    },
  }
)

// Start polling
startPolling({ reportId: '123' })

// Stop polling
stopPolling()
```

### Cursor-Based Pagination

```typescript
import { useIntegrationWithCursorLoadMore } from '#integration/composables'

const {
  data,
  loading,
  isEmpty,
  cursor,
  canLoadMore,
  fetchData,
  loadMore,
  resetAndFetch,
} = useIntegrationWithCursorLoadMore<T, QueryParams>({
  query: ref({}),
  onFetch: async (params) => {
    return await api.list(params)
  },
  withLoading: true,
  immediate: true,
})
```

### Parallel Data Fetching

```vue
<script setup lang="ts">
// Multiple independent fetches
const { data: user } = useAsyncData('user', () => userApi.getCurrentUser())
const { data: settings } = useAsyncData('settings', () => settingsApi.getSettings())
const { data: preferences } = useAsyncData('preferences', () => preferencesApi.getPreferences())

// All run in parallel automatically
</script>
```

### Dependent Queries

```vue
<script setup lang="ts">
// Fetch user first
const { data: user } = useAsyncData('user', () => userApi.getUser(userId.value))

// Then fetch user's settings (waits for user)
const { data: settings } = useAsyncData(
  'user-settings',
  () => settingsApi.getUserSettings(user.value.id),
  {
    watch: [user], // Refetch when user changes
    server: false,
  }
)
</script>
```

---

## State Management

### Server-Side State (useState)

```typescript
// Shared state across components (SSR-safe)
const siteData = useState<Site>('site-data', () => null)

// Use in API service or composable
async function fetchSite() {
  const data = await api.getSite()
  siteData.value = data
  return data
}
```

### Client-Only State (ref)

```typescript
// Client-only reactive state
const localData = ref<Data>(null)

// Use for client-only operations
async function fetchLocal() {
  localData.value = await api.getData()
}
```

**When to use:**
- `useState` - Shared state, SSR data, needs to persist across navigation
- `ref` - Component-local state, client-only operations

---

## Complete Examples

### Example 1: Simple Entity Fetch

```vue
<script setup lang="ts">
import { useApiSiteDetails } from '#site-studio/composables'

const route = useRoute('site-studio-id')
const siteId = computed(() => route.params.id as string)
const { getSiteDetails } = useApiSiteDetails()

const {
  data: siteDetail,
  error: siteError,
  pending: sitePending,
  refresh: refreshSiteDetail,
} = useAsyncData(
  'site-details',
  () => getSiteDetails(unref(siteId)),
  {
    server: false,
    watch: [siteId],
  }
)

whenError(siteError, () => navigateTo('/site-studio'))
</script>

<template>
  <PageBody v-loading="sitePending">
    <div v-if="siteDetail?.data">
      <h1>{{ siteDetail.data.site_name }}</h1>
    </div>
  </PageBody>
</template>
```

### Example 2: Paginated List

```vue
<script setup lang="ts">
import { useApiJobDataPackages } from '#datafeed/composables'

const {
  fetchMoreData: fetchMoreFiles,
  fetchData: fetchFiles,
  listData: files,
  listQuery,
  listLoading,
  canLoadMore,
} = useListData('job-data-packages-file-list', {
  query: ['page', 'per_page', 'keyword'],
  defaultLimit: 20,
  server: false,
  lazy: true,
  immediate: false,
  async onFetch(query) {
    const response = await useApiJobDataPackages().list({
      keyword: query.keyword,
      page: query.page,
      per_page: query.per_page,
    })
    return response.data_feed_files
  },
})

onMounted(() => {
  fetchFiles()
})
</script>

<template>
  <div>
    <input
      v-model="listQuery.keyword"
      placeholder="Search files..."
    />
    <div v-for="file in files" :key="file.uuid">
      {{ file.name }}
    </div>
    <button
      v-if="canLoadMore"
      :disabled="listLoading"
      @click="fetchMoreFiles"
    >
      Load More
    </button>
  </div>
</template>
```

### Example 3: Mutation with Error Handling

```vue
<script setup lang="ts">
import { useApiSiteDetails } from '#site-studio/composables'

const { updateSite, getSiteDetails } = useApiSiteDetails()
const siteId = computed(() => route.params.id as string)
const isLoading = ref(false)
const serverErrors = ref<Record<string, number>>({})

const {
  data: siteDetail,
  refresh: refreshSiteDetail,
} = useAsyncData('site-details', () => getSiteDetails(unref(siteId)), {
  server: false,
})

async function handleSave() {
  return withCatch(
    async () => {
      isLoading.value = true
      OlLoading()

      await updateSite(siteId.value, formData.value)

      OlNotify.success?.({
        message: $gettext('Site updated successfully'),
      })

      await refreshSiteDetail()
    },
    {
      notify: false,
      onError: (error: any) => {
        const errors = error.data?.errors || []
        if (errors.length) {
          serverErrors.value = errors.reduce((acc, err) => {
            acc[err.field] = err.code
            return acc
          }, {})
        } else {
          OlNotify.error?.({
            message: $gettext('Failed to update site'),
          })
        }
      },
      onFinally: () => {
        isLoading.value = false
        OlLoading()?.close()
      },
    }
  )
}
</script>
```

---

## Best Practices

### 1. API Service Organization

- One API service file per feature area
- Use composable wrapper (`useApiX`) for consistency
- Type all parameters and return values
- Use constants for endpoints

### 2. Data Fetching

- Use `useAsyncData` for single fetches
- Use `useListData` for paginated lists
- Set `server: false` for authenticated requests
- Use `watch` option for reactive refetches

### 3. Error Handling

- Always use `withCatch` for mutations
- Use `whenError` for `useAsyncData` errors
- Provide user-friendly error messages
- Handle server validation errors separately

### 4. Loading States

- Use `pending` from `useAsyncData` for loading
- Use `listLoading` from `useListData` for lists
- Use `OlLoading()` for global loading overlay
- Disable buttons during mutations

### 5. State Management

- Use `useState` for shared/SSR state
- Use `ref` for component-local state
- Clear state on component unmount when needed

### 6. Type Safety

- Type all API responses
- Use TypeScript interfaces for data structures
- Type composable return values
- Avoid `any` types

---

## Summary

**Modern Data Fetching Recipe:**

1. **Create API Service**: `areas/X/app/api/XApi.ts` using `$http`
2. **Create Composable**: `areas/X/app/composables/useApiX.ts` wrapper
3. **Use useAsyncData**: For single entity fetches
4. **Use useListData**: For paginated lists with search
5. **Route Format**: Direct paths, no `/api/` prefix
6. **Mutations**: Wrap in `withCatch` with error handling
7. **Error Handling**: `whenError` for queries, `withCatch` for mutations
8. **Type Safety**: Type all parameters and returns

**See Also:**
- Nuxt 3 Docs: [Data Fetching](https://nuxt.com/docs/getting-started/data-fetching)
- Nuxt 3 Docs: [useAsyncData](https://nuxt.com/docs/api/composables/use-async-data)
- Project: `areas/common/app/composables/useListData.ts` - List data implementation
- Project: `areas/common/app/utils/error.ts` - Error handling utilities
