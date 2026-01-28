# Common Patterns

Frequently used patterns for forms, authentication, data fetching, dialogs, and other common UI elements in Vue/Nuxt.

---

## Authentication

### Getting Current User

```vue
<script setup lang="ts">
const { loggedInUser } = useLoggedIn()

// Available properties:
// - loggedInUser.value.id: number
// - loggedInUser.value.email: string
// - loggedInUser.value.name: string
// - loggedInUser.value.is_staff: boolean
// - loggedInUser.value.is_superuser: boolean
</script>

<template>
  <div>
    <p>Logged in as: {{ loggedInUser?.email }}</p>
    <p>Name: {{ loggedInUser?.name }}</p>
  </div>
</template>
```

### Using Auth State

```vue
<script setup lang="ts">
const { data: authData } = useAuthState()
const { isAuthenticated, status } = useAuth()

// Check authentication status
if (isAuthenticated.value) {
  // User is authenticated
}
</script>
```

**NEVER make direct API calls for auth** - always use `useAuth`, `useLoggedIn`, or `useAuthState` composables.

---

## Forms with OlForm

### Basic Form with Validation

```vue
<script setup lang="ts">
import { z } from 'zod'
import type { FormInstance } from 'element-plus'

const formRef = ref<FormInstance>()

// Zod schema for validation
const formSchema = z.object({
  username: z.string().min(3, 'Username must be at least 3 characters'),
  email: z.string().email('Invalid email address'),
  age: z.number().min(18, 'Must be 18 or older'),
})

type FormData = z.infer<typeof formSchema>

const formModel = reactive<FormData>({
  username: '',
  email: '',
  age: 18,
})

async function handleSubmit() {
  if (!formRef.value) return

  await formRef.value.validate(async (isValid) => {
    if (isValid) {
      try {
        // Parse with Zod for type safety
        const data = formSchema.parse(formModel)
        
        // Submit form
        await api.submitForm(data)
        
        OlNotify.success({
          message: 'Form submitted successfully',
        })
      }
      catch (error) {
        OlNotify.error({
          message: 'Failed to submit form',
        })
      }
    }
  })
}
</script>

<template>
  <OlForm
    ref="formRef"
    :model="formModel"
    :schema="formSchema"
  >
    <OlFormItem
      label="Username"
      prop="username"
    >
      <ElInput v-model="formModel.username" />
    </OlFormItem>

    <OlFormItem
      label="Email"
      prop="email"
    >
      <ElInput
        v-model="formModel.email"
        type="email"
      />
    </OlFormItem>

    <OlFormItem
      label="Age"
      prop="age"
    >
      <ElInputNumber v-model="formModel.age" />
    </OlFormItem>

    <ElButton
      type="primary"
      @click="handleSubmit"
    >
      Submit
    </ElButton>
  </OlForm>
</template>
```

### Form with Rules (Element Plus)

```vue
<script setup lang="ts">
import type { FormInstance, FormRules } from 'element-plus'

const formRef = ref<FormInstance>()

const formModel = reactive({
  username: '',
  email: '',
})

const rules: FormRules = {
  username: [
    { required: true, message: 'Username is required', trigger: 'blur' },
    { min: 3, max: 20, message: 'Length should be 3 to 20', trigger: 'blur' },
  ],
  email: [
    { required: true, message: 'Email is required', trigger: 'blur' },
    { type: 'email', message: 'Invalid email address', trigger: 'blur' },
  ],
}

async function handleSubmit() {
  if (!formRef.value) return

  await formRef.value.validate((isValid) => {
    if (isValid) {
      // Submit form
    }
  })
}
</script>

<template>
  <OlForm
    ref="formRef"
    :model="formModel"
    :rules="rules"
  >
    <OlFormItem
      label="Username"
      prop="username"
    >
      <ElInput v-model="formModel.username" />
    </OlFormItem>

    <OlFormItem
      label="Email"
      prop="email"
    >
      <ElInput v-model="formModel.email" />
    </OlFormItem>
  </OlForm>
</template>
```

---

## Dialog and Drawer Components

### Standard Drawer Pattern

```vue
<script setup lang="ts">
const isDrawerVisible = ref(false)

function openDrawer() {
  isDrawerVisible.value = true
}

function closeDrawer() {
  isDrawerVisible.value = false
}

function handleConfirm() {
  // Handle confirmation logic
  closeDrawer()
}
</script>

<template>
  <OlDrawer
    v-model:value="isDrawerVisible"
    title="Drawer Title"
    :width="600"
  >
    <template #header>
      <div class=":uno: flex items-center gap-2">
        <i class=":uno: i-ol-icon icon:text-primary" />
        <span>Drawer Title</span>
      </div>
    </template>

    <div class=":uno: p-4">
      <!-- Content here -->
    </div>

    <template #footer>
      <div class=":uno: flex justify-end gap-2">
        <ElButton @click="closeDrawer">
          Cancel
        </ElButton>
        <ElButton
          type="primary"
          @click="handleConfirm"
        >
          Confirm
        </ElButton>
      </div>
    </template>
  </OlDrawer>
</template>
```

### Dialog Service Pattern

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

    // User confirmed, proceed with delete
    await api.deleteItem()
    
    OlNotify.success({
      message: 'Item deleted successfully',
    })
  }
  catch {
    // User cancelled
  }
}
</script>
```

### Drawer with Confirmation

```vue
<script setup lang="ts">
const isDrawerVisible = ref(false)

function handleBeforeClose(done: () => void) {
  // Custom confirmation logic
  OlDialogService({
    type: 'warning',
    title: 'Unsaved Changes',
    body: 'You have unsaved changes. Are you sure you want to close?',
  })
    .then(() => done())
    .catch(() => {
      // User cancelled
    })
}
</script>

<template>
  <OlDrawer
    v-model:value="isDrawerVisible"
    :before-close="handleBeforeClose"
    confirmation
  >
    <!-- Drawer content -->
  </OlDrawer>
</template>
```

---

## Data Fetching Patterns

### Using useAsyncData

```vue
<script setup lang="ts">
import { useApiPosts } from '#posts/composables'

const route = useRoute('posts-id')
const postId = computed(() => route.params.id as string)
const { getPost } = useApiPosts()

const {
  data: post,
  error: postError,
  pending: postPending,
  refresh: refreshPost,
} = useAsyncData(
  `post-${postId.value}`,
  () => getPost(unref(postId)),
  {
    server: false,
    watch: [postId],
  }
)

whenError(postError, () => {
  OlNotify.error({
    message: 'Failed to load post',
  })
})
</script>

<template>
  <div v-loading="postPending">
    <div v-if="post">
      <h1>{{ post.title }}</h1>
      <p>{{ post.content }}</p>
    </div>
  </div>
</template>
```

### Using useListData for Paginated Lists

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
    <ElInput
      v-model="listQuery.keyword"
      placeholder="Search files..."
    />
    
    <div v-for="file in files" :key="file.uuid">
      {{ file.name }}
    </div>
    
    <ElButton
      v-if="canLoadMore"
      :loading="listLoading"
      @click="fetchMoreFiles"
    >
      Load More
    </ElButton>
  </div>
</template>
```

### Mutation Pattern with Refresh

```vue
<script setup lang="ts">
import { useApiPosts } from '#posts/composables'

const route = useRoute('posts-id')
const postId = computed(() => route.params.id as string)
const { getPost, updatePost } = useApiPosts()
const isLoading = ref(false)

const {
  data: post,
  refresh: refreshPost,
} = useAsyncData(
  `post-${postId.value}`,
  () => getPost(unref(postId)),
  { server: false }
)

async function handleUpdate() {
  isLoading.value = true
  
  try {
    await updatePost(unref(postId), {
      title: post.value.title,
      content: post.value.content,
    })
    
    // Refresh the data
    await refreshPost()
    
    OlNotify.success({
      message: 'Post updated successfully',
    })
  }
  catch (error) {
    OlNotify.error({
      message: 'Failed to update post',
    })
  }
  finally {
    isLoading.value = false
  }
}
</script>
```

---

## State Management Patterns

### Pinia Stores for Global State

```typescript
// stores/account.ts
import { defineStore } from 'pinia'

export const useAccountStore = defineStore('account', () => {
  const account = ref<Account | null>(null)
  
  async function fetchAccount() {
    const response = await api.getAccount()
    account.value = response
  }
  
  return {
    account,
    fetchAccount,
  }
})
```

### Using Stores in Components

```vue
<script setup lang="ts">
const { account } = storeToRefs(useAccountStore())

// Access reactive store state
const accountName = computed(() => account.value?.name)
</script>

<template>
  <div v-if="account">
    {{ account.name }}
  </div>
</template>
```

### Local Component State (ref)

Use `ref` for **component-local state**:
- Form inputs
- Modal/drawer visibility
- Selected items
- Temporary UI flags

```vue
<script setup lang="ts">
// ✅ CORRECT - ref for local state
const isDrawerOpen = ref(false)
const selectedItem = ref<string | null>(null)
const formData = ref({
  name: '',
  email: '',
})
</script>
```

### Shared State (useState)

Use `useState` for **shared state across components** (SSR-safe):

```vue
<script setup lang="ts">
// Shared state that persists across navigation
const siteData = useState<Site>('site-data', () => null)

async function fetchSite() {
  const data = await api.getSite()
  siteData.value = data
}
</script>
```

**When to use:**
- `ref` - Component-local state, client-only operations
- `useState` - Shared state, SSR data, needs to persist across navigation
- Pinia stores - Global application state, complex state logic

---

## Notifications

### Success Messages

```vue
<script setup lang="ts">
function handleSave() {
  OlNotify.success({
    message: 'Data saved successfully',
  })
}
</script>
```

### Error Messages

```vue
<script setup lang="ts">
function handleError() {
  OlNotify.error({
    message: 'An error occurred',
  })
}
</script>
```

### Notifications with Options

```vue
<script setup lang="ts">
OlNotify({
  type: 'success',
  message: 'Operation completed',
  duration: 3000,
  position: 'top-right',
})
</script>
```

---

## Common Component Patterns

### Loading States

```vue
<script setup lang="ts">
const isLoading = ref(false)

async function handleAction() {
  isLoading.value = true
  try {
    await api.performAction()
  }
  finally {
    isLoading.value = false
  }
}
</script>

<template>
  <div v-loading="isLoading">
    <!-- Content -->
  </div>
  
  <!-- Or with button -->
  <ElButton
    :loading="isLoading"
    @click="handleAction"
  >
    Submit
  </ElButton>
</template>
```

### Conditional Rendering

```vue
<template>
  <div v-if="pending" class=":uno: p-4">
    Loading...
  </div>
  <div v-else-if="error" class=":uno: p-4 text-danger">
    Error: {{ error.message }}
  </div>
  <div v-else class=":uno: p-4">
    {{ data }}
  </div>
</template>
```

### Computed Properties

```vue
<script setup lang="ts">
const items = ref([1, 2, 3, 4, 5])

const filteredItems = computed(() => {
  return items.value.filter(item => item > 2)
})

const itemCount = computed(() => items.value.length)
</script>
```

### Watchers

```vue
<script setup lang="ts">
const searchQuery = ref('')
const results = ref([])

watch(searchQuery, async (newQuery) => {
  if (newQuery.length > 2) {
    results.value = await api.search(newQuery)
  }
}, { debounce: 300 })
</script>
```

---

## Route Navigation

### Programmatic Navigation

```vue
<script setup lang="ts">
import { navigateTo } from '#imports'

function handleNavigate() {
  navigateTo('/posts')
}

function handleNavigateWithParams() {
  navigateTo({
    name: 'posts-id',
    params: { id: '123' },
    query: { tab: 'details' },
  })
}
</script>
```

### Using Router

```vue
<script setup lang="ts">
const router = useRouter()

function handleReplace() {
  router.replace({
    path: '/posts',
    query: { updated: true },
  })
}
</script>
```

---

## Summary

**Common Patterns:**
- ✅ `useLoggedIn()` for current user (id, email, name, is_staff)
- ✅ `OlForm` with Zod schema or Element Plus rules for forms
- ✅ `OlDrawer` and `OlDialog` with `v-model:value` for modals
- ✅ `useAsyncData` for single data fetches
- ✅ `useListData` for paginated lists
- ✅ Pinia stores with `storeToRefs` for global state
- ✅ `ref` for component-local state
- ✅ `useState` for shared SSR-safe state
- ✅ `OlNotify` for success/error messages
- ✅ `navigateTo()` for programmatic navigation

**Form Validation:**
- Use Zod schemas with `OlForm` for type-safe validation
- Use Element Plus rules for simple validation
- Always validate before submitting

**Data Fetching:**
- `useAsyncData` for single entity fetches
- `useListData` for paginated lists
- Always handle loading and error states

**State Management:**
- `ref` - Component-local state
- `useState` - Shared SSR-safe state
- Pinia stores - Global application state

**See Also:**
- [data-fetching.md](data-fetching.md) - Detailed data fetching patterns
- [component-patterns.md](component-patterns.md) - Component structure
- [routing-guide.md](routing-guide.md) - Navigation patterns
- [styling-guide.md](styling-guide.md) - Styling with UnoCSS
