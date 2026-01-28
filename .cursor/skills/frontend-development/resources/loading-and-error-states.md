# Loading & Error States

**CRITICAL**: Proper loading and error state handling prevents layout shift and provides better user experience.

---

## ⚠️ CRITICAL RULE: Never Use Early Returns

### The Problem

```vue
<!-- ❌ NEVER DO THIS - Early return with loading spinner -->
<script setup lang="ts">
const { data, pending } = useAsyncData('users', () => api.getUsers())

// WRONG: This causes layout shift and poor UX
</script>

<template>
  <div v-if="pending">
    <OlSpinner />
  </div>
  <div v-else>
    <Content :data="data" />
  </div>
</template>
```

**Why this is bad:**
1. **Layout Shift**: Content position jumps when loading completes
2. **CLS (Cumulative Layout Shift)**: Poor Core Web Vital score
3. **Jarring UX**: Page structure changes suddenly
4. **Lost Scroll Position**: User loses place on page

### The Solutions

**Option 1: v-loading Directive (PREFERRED)**

```vue
<script setup lang="ts">
const { data, pending } = useAsyncData('users', () => api.getUsers())
</script>

<template>
  <div v-loading="pending">
    <div v-if="data">
      <Content :data="data" />
    </div>
  </div>
</template>
```

**Option 2: Suspense for Async Components**

```vue
<script setup lang="ts">
const LazyHeavyComponent = defineAsyncComponent(() => 
  import('#feature/components/heavy-component.vue')
)
</script>

<template>
  <Suspense>
    <template #default>
      <LazyHeavyComponent />
    </template>
    <template #fallback>
      <div class=":uno: flex justify-center p-4">
        <OlSpinner />
      </div>
    </template>
  </Suspense>
</template>
```

---

## v-loading Directive

### What It Does

- Shows loading indicator overlay on element
- Prevents layout shift (content area reserved)
- Prevents interaction while loading
- Consistent loading experience across app

### Basic Usage

```vue
<script setup lang="ts">
const isLoading = ref(false)
const { data, pending } = useAsyncData('users', () => api.getUsers())
</script>

<template>
  <!-- Direct loading state -->
  <div v-loading="isLoading">
    <Content />
  </div>

  <!-- With useAsyncData pending -->
  <div v-loading="pending">
    <div v-if="data">
      {{ data }}
    </div>
  </div>

  <!-- On specific element -->
  <PageCard v-loading="pending">
    <Content />
  </PageCard>
</template>
```

### With Custom Loading Class

```vue
<template>
  <div
    v-loading="pending"
    class=":uno: all-[.el-loading-mask]:!h-full"
  >
    <Content />
  </div>
</template>
```

### Multiple Loading States

```vue
<script setup lang="ts">
const { data: users, pending: usersPending } = useAsyncData('users', () => api.getUsers())
const { data: roles, pending: rolesPending } = useAsyncData('roles', () => api.getRoles())

const isLoading = computed(() => usersPending.value || rolesPending.value)
</script>

<template>
  <div v-loading="isLoading">
    <UserList v-if="users" :users="users" />
    <RoleList v-if="roles" :roles="roles" />
  </div>
</template>
```

---

## Suspense for Async Components

### When to Use Suspense

- Lazy loaded components (`defineAsyncComponent`)
- Components that use `useAsyncData` with `lazy: false`
- Independent sections that can load separately

### Basic Usage

```vue
<script setup lang="ts">
const LazyUserProfile = defineAsyncComponent(() => 
  import('#user/components/user-profile.vue')
)
</script>

<template>
  <Suspense>
    <LazyUserProfile :user-id="userId" />
  </Suspense>
</template>
```

### With Custom Fallback

```vue
<template>
  <Suspense>
    <template #default>
      <LazyHeavyComponent />
    </template>
    <template #fallback>
      <div class=":uno: flex items-center justify-center p-8">
        <OlSpinner />
        <span class=":uno: ml-4 text-grey-600">Loading...</span>
      </div>
    </template>
  </Suspense>
</template>
```

### Multiple Suspense Boundaries

```vue
<template>
  <div class=":uno: grid grid-cols-3 gap-4">
    <Suspense>
      <HeaderSection />
    </Suspense>

    <Suspense>
      <MainContent />
    </Suspense>

    <Suspense>
      <Sidebar />
    </Suspense>
  </div>
</template>
```

**Benefits:**
- Each section loads independently
- User sees partial content sooner
- Better perceived performance

---

## OlLoading Service

### Full-Screen Loading

Use `OlLoading()` service for full-screen loading during mutations:

```vue
<script setup lang="ts">
const isLoading = ref(false)

async function handleSave() {
  isLoading.value = true
  const loadingInstance = OlLoading()
  
  try {
    await api.save(data)
    OlNotify.success({
      message: 'Saved successfully',
    })
  }
  catch (error) {
    OlNotify.error({
      message: 'Failed to save',
    })
  }
  finally {
    isLoading.value = false
    loadingInstance?.close()
  }
}
</script>
```

### With withCatch

```vue
<script setup lang="ts">
import { withCatch } from '#common/utils/error'

async function handleSave() {
  return withCatch(
    async () => {
      OlLoading()
      const response = await api.save(data)
      
      OlNotify.success({
        message: 'Saved successfully',
      })
      
      return response
    },
    {
      notify: false,
      onError: (error) => {
        OlNotify.error({
          message: 'Failed to save',
        })
      },
      onFinally: () => {
        OlLoading()?.close()
      },
    }
  )
}
</script>
```

### Target-Specific Loading

```vue
<script setup lang="ts">
const drawerRef = useTemplateRef<InstanceType<typeof OlDrawer>>('drawerRef')

function startLoading() {
  const drawerEl = drawerRef.value?.getDrawerId()
  const target = document.getElementById(drawerEl)
  
  if (target) {
    OlLoading({
      target,
      lock: true,
    })
  }
}
</script>
```

---

## Error Handling

### OlNotify for User Feedback (REQUIRED)

**ALWAYS use `OlNotify`** for user notifications - Project standard

```vue
<script setup lang="ts">
async function handleAction() {
  try {
    await api.doSomething()
    OlNotify.success({
      message: 'Operation completed successfully',
    })
  }
  catch (error) {
    OlNotify.error({
      message: 'Operation failed',
    })
  }
}
</script>
```

**Available Methods:**
- `OlNotify.success({ message: '...' })` - Green success message
- `OlNotify.error({ message: '...' })` - Red error message
- `OlNotify.warning({ message: '...' })` - Orange warning message
- `OlNotify.info({ message: '...' })` - Blue info message

### whenError for useAsyncData

Automatically watch and handle errors from `useAsyncData`:

```vue
<script setup lang="ts">
import { whenError } from '#common/utils/error'

const { data, error } = useAsyncData('users', () => api.getUsers())

// Automatically handle errors
whenError(error, (err) => {
  // Custom error handling
  console.error('Failed to load users:', err)
  navigateTo('/error-page')
})

// Or use default behavior (shows error notification)
whenError(error)
</script>
```

### withCatch for Mutations

Wrap async operations with error handling:

```vue
<script setup lang="ts">
import { withCatch } from '#common/utils/error'

async function handleUpdate() {
  return withCatch(
    async () => {
      OlLoading()
      const response = await api.update(id, data)
      
      OlNotify.success({
        message: 'Updated successfully',
      })
      
      await refreshData()
      return response
    },
    {
      notify: false, // Handle errors manually
      onError: (error) => {
        // Custom error handling
        const errors = error.data?.errors || []
        if (errors.length) {
          setServerErrors(errors)
        }
        else {
          OlNotify.error({
            message: 'Failed to update',
          })
        }
      },
      onFinally: () => {
        OlLoading()?.close()
      },
    }
  )
}
</script>
```

### Error Dialog Service

For critical errors that need user confirmation:

```vue
<script setup lang="ts">
async function handleDelete() {
  try {
    await OlDialogService({
      type: 'danger',
      title: 'Confirm Delete',
      body: 'Are you sure you want to delete this item?',
      textConfirm: 'Delete',
      textCancel: 'Cancel',
    })
    
    await api.delete(id)
    OlNotify.success({
      message: 'Deleted successfully',
    })
  }
  catch {
    // User cancelled
  }
}
</script>
```

---

## Error Pages

### Nuxt Error Page

Nuxt automatically handles errors with `error.vue`:

```vue
<!-- app/error.vue -->
<script setup lang="ts">
import type { NuxtError } from '#app'

const props = defineProps<{
  error: NuxtError & { url?: string }
}>()

const { $gettext } = useLocale()

const title = computed(() => {
  if (props.error.statusCode === 404) {
    return $gettext('Page Not Found')
  }
  if (props.error.statusCode === 403) {
    return $gettext('Access Denied')
  }
  return $gettext('Something went wrong')
})
</script>

<template>
  <NuxtLayout :name="error?.statusCode === 404 ? false : 'blank'">
    <PageError
      :title="title"
      :description="$gettext('An error occurred')"
    />
  </NuxtLayout>
</template>
```

### Custom Error Components

```vue
<script setup lang="ts">
const hasError = ref(false)
const errorMessage = ref('')

function handleError(error: Error) {
  hasError.value = true
  errorMessage.value = error.message
}
</script>

<template>
  <div v-if="hasError" class=":uno: p-4">
    <OlEmptyResult>
      <template #icon>
        <i class=":uno: i-ol-alert-caution !text-xl icon:text-grey" />
      </template>
      <template #info>
        <div>{{ $gettext('Something went wrong.') }}</div>
        <div>{{ $gettext('Please try again.') }}</div>
      </template>
    </OlEmptyResult>
  </div>
  <div v-else>
    <Content />
  </div>
</template>
```

---

## Complete Examples

### Example 1: Component with useAsyncData

```vue
<script setup lang="ts">
import { whenError } from '#common/utils/error'

const route = useRoute('users-id')
const userId = computed(() => route.params.id as string)

const {
  data: user,
  error: userError,
  pending: userPending,
  refresh: refreshUser,
} = useAsyncData(
  `user-${userId.value}`,
  () => useApiUsers().getUser(unref(userId)),
  {
    server: false,
    watch: [userId],
  }
)

// Handle errors automatically
whenError(userError, () => {
  navigateTo('/users')
})
</script>

<template>
  <PageCard v-loading="userPending">
    <div v-if="user">
      <h1>{{ user.name }}</h1>
      <p>{{ user.email }}</p>
    </div>
  </PageCard>
</template>
```

### Example 2: Mutation with Loading and Error Handling

```vue
<script setup lang="ts">
import { withCatch } from '#common/utils/error'

const isLoading = ref(false)
const formRef = useTemplateRef<InstanceType<typeof OlForm>>('formRef')

const {
  data: user,
  refresh: refreshUser,
} = useAsyncData('user', () => useApiUsers().getUser(userId.value), {
  server: false,
})

async function handleSave() {
  if (!formRef.value) return

  await formRef.value.validate(async (isValid) => {
    if (isValid) {
      await withCatch(
        async () => {
          isLoading.value = true
          OlLoading()

          await useApiUsers().updateUser(userId.value, formData.value)
          
          OlNotify.success({
            message: 'User updated successfully',
          })
          
          await refreshUser()
        },
        {
          notify: false,
          onError: (error) => {
            const errors = error.data?.errors || []
            if (errors.length) {
              // Set form field errors
              formRef.value?.setFields(errors)
            }
            else {
              OlNotify.error({
                message: 'Failed to update user',
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
  })
}
</script>

<template>
  <OlForm ref="formRef" :model="formData">
    <OlFormItem label="Name" prop="name">
      <ElInput v-model="formData.name" />
    </OlFormItem>

    <ElButton
      type="primary"
      :loading="isLoading"
      @click="handleSave"
    >
      Save
    </ElButton>
  </OlForm>
</template>
```

### Example 3: List with Loading and Error States

```vue
<script setup lang="ts">
import { whenError } from '#common/utils/error'

const {
  listData: users,
  listLoading,
  listQuery,
  fetchData,
} = useListData('users', {
  query: ['page', 'per_page', 'keyword'],
  defaultLimit: 20,
  server: false,
  async onFetch(query) {
    const response = await useApiUsers().list({
      keyword: query.keyword,
      page: query.page,
      per_page: query.per_page,
    })
    return response.users
  },
})

const { error } = useAsyncData('users-list', () => fetchData())

whenError(error)
</script>

<template>
  <div v-loading="listLoading">
    <div v-if="users && users.length > 0">
      <div
        v-for="user in users"
        :key="user.id"
        class=":uno: p-4 border border-grey-200 rounded"
      >
        {{ user.name }}
      </div>
    </div>
    <OlEmptyResult v-else>
      <template #info>
        {{ $gettext('No users found') }}
      </template>
    </OlEmptyResult>
  </div>
</template>
```

### Example 4: Suspense with Async Component

```vue
<script setup lang="ts">
const LazyUserDetailDrawer = defineAsyncComponent(() => 
  import('#user/components/user-management/user-detail/user-detail-drawer.vue')
)

const isDrawerVisible = ref(false)
const userDetail = ref<UserDetail>()
</script>

<template>
  <div>
    <ElButton @click="isDrawerVisible = true">
      Open Drawer
    </ElButton>

    <Suspense v-if="isDrawerVisible">
      <template #default>
        <LazyUserDetailDrawer
          :user-detail="userDetail"
          @closed="isDrawerVisible = false"
        />
      </template>
      <template #fallback>
        <OlDrawer
          v-model:value="true"
          title="Loading..."
        >
          <div class=":uno: flex justify-center p-8">
            <OlSpinner />
          </div>
        </OlDrawer>
      </template>
    </Suspense>
  </div>
</template>
```

---

## Loading State Anti-Patterns

### ❌ What NOT to Do

```vue
<!-- ❌ NEVER - Early return -->
<template>
  <div v-if="pending">
    <OlSpinner />
  </div>
  <div v-else>
    <Content />
  </div>
</template>

<!-- ❌ NEVER - Conditional layout changes -->
<template>
  <div v-if="pending" class=":uno: h-20">
    <OlSpinner />
  </div>
  <div v-else class=":uno: h-96">
    <Content />
  </div>
</template>

<!-- ❌ NEVER - Replace content with spinner -->
<template>
  <div>
    <OlSpinner v-if="pending" />
    <Content v-else />
  </div>
</template>
```

### ✅ What TO Do

```vue
<!-- ✅ BEST - v-loading directive -->
<template>
  <div v-loading="pending">
    <Content v-if="data" :data="data" />
  </div>
</template>

<!-- ✅ GOOD - Suspense for async components -->
<template>
  <Suspense>
    <LazyComponent />
  </Suspense>
</template>

<!-- ✅ OK - Skeleton with same layout -->
<template>
  <div class=":uno: h-96">
    <div v-if="pending" class=":uno: animate-pulse bg-grey-200 rounded" />
    <Content v-else :data="data" />
  </div>
</template>
```

---

## Skeleton Loading (Alternative)

### Custom Skeleton Component

```vue
<script setup lang="ts">
const { data, pending } = useAsyncData('user', () => api.getUser())
</script>

<template>
  <div class=":uno: p-4">
    <div v-if="pending" class=":uno: space-y-4">
      <div class=":uno: h-8 w-48 bg-grey-200 rounded animate-pulse" />
      <div class=":uno: h-4 w-full bg-grey-200 rounded animate-pulse" />
      <div class=":uno: h-4 w-3/4 bg-grey-200 rounded animate-pulse" />
    </div>
    <div v-else>
      <h1 class=":uno: text-2xl font-semibold">{{ data.name }}</h1>
      <p>{{ data.description }}</p>
    </div>
  </div>
</template>
```

**Key**: Skeleton must have **same layout** as actual content (no shift)

---

## Error Handling Patterns

### API Field Errors

```vue
<script setup lang="ts">
import { withCatch } from '#common/utils/error'

const formRef = useTemplateRef<InstanceType<typeof OlForm>>('formRef')
const serverErrors = ref<Record<string, number>>({})

async function handleSubmit() {
  await withCatch(
    async () => {
      await api.submit(formData.value)
      OlNotify.success({ message: 'Submitted successfully' })
    },
    {
      notify: false,
      onError: (error) => {
        const errors = error.data?.errors || []
        if (errors.length) {
          // Set form field errors
          serverErrors.value = errors.reduce((acc, err) => {
            acc[err.field] = err.message
            return acc
          }, {} as Record<string, string>)
          
          // Mark form fields as invalid
          formRef.value?.setFields(errors)
        }
        else {
          OlNotify.error({
            message: 'Failed to submit',
          })
        }
      },
    }
  )
}
</script>
```

### Error Recovery

```vue
<script setup lang="ts">
const { data, error, refresh } = useAsyncData('data', () => api.getData())

whenError(error, () => {
  // Error already handled by whenError
})

function handleRetry() {
  refresh()
}
</script>

<template>
  <div v-if="error" class=":uno: p-4 text-center">
    <OlEmptyResult>
      <template #icon>
        <i class=":uno: i-ol-alert-caution" />
      </template>
      <template #info>
        <div>{{ $gettext('Failed to load data') }}</div>
        <ElButton @click="handleRetry">
          {{ $gettext('Try Again') }}
        </ElButton>
      </template>
    </OlEmptyResult>
  </div>
  <div v-else v-loading="pending">
    <Content :data="data" />
  </div>
</template>
```

---

## Summary

**Loading States:**
- ✅ **PREFERRED**: `v-loading` directive (prevents layout shift)
- ✅ **GOOD**: `Suspense` for async components
- ✅ **OK**: Skeleton with same layout
- ✅ **FULL-SCREEN**: `OlLoading()` service for mutations
- ❌ **NEVER**: Early returns or conditional layout changes

**Error Handling:**
- ✅ **ALWAYS**: `OlNotify` for user feedback
- ✅ **useAsyncData**: `whenError(error)` to watch errors
- ✅ **Mutations**: `withCatch()` wrapper
- ✅ **Critical**: `OlDialogService` for confirmations
- ✅ **Pages**: Nuxt `error.vue` for route errors

**Key Principles:**
1. Never use early returns for loading states
2. Always reserve space for content (prevent layout shift)
3. Use `v-loading` on containers, not conditionally rendered elements
4. Always handle errors with user feedback
5. Use `whenError` for automatic error watching
6. Use `withCatch` for mutations with proper cleanup

**See Also:**
- [component-patterns.md](component-patterns.md) - Component structure
- [data-fetching.md](data-fetching.md) - `useAsyncData` and `useListData` details
- [common-patterns.md](common-patterns.md) - Common UI patterns
