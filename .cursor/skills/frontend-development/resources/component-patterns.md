# Component Patterns

Modern Vue 3 component architecture for Nuxt 3 applications emphasizing type safety, lazy loading, composables, and clear separation of concerns.

---

## Vue 3 Composition API Pattern (PREFERRED)

### Why `<script setup>`

All components use `<script setup lang="ts">` for:
- Explicit type safety for props and emits
- Consistent component signatures
- Clear prop/emit interface documentation
- Better IDE autocomplete
- Automatic exports (no need for `export default`)
- Better performance (compile-time optimization)

### Basic Pattern

```vue
<script setup lang="ts">
/**
 * Component description
 * What it does, when to use it
 */

// 1. Define component name
defineOptions({
  name: 'MyComponent',
})

// 2. Define props with TypeScript
interface Props {
  /** User ID to display */
  userId: number
  /** Optional callback when action occurs */
  onAction?: () => void
  /** Display mode */
  mode?: 'view' | 'edit'
}

const props = withDefaults(defineProps<Props>(), {
  mode: 'view',
})

// 3. Define emits
interface Emits {
  (e: 'action'): void
  (e: 'update', value: string): void
}

const emit = defineEmits<Emits>()

// 4. Component logic
const message = computed(() => `User: ${props.userId}`)

function handleClick() {
  emit('action')
  props.onAction?.()
}
</script>

<template>
  <div class=":uno: p-4">
    {{ message }}
  </div>
</template>
```

**Key Points:**
- `defineOptions` for component name (useful for debugging)
- Props interface defined with JSDoc comments
- `withDefaults` for default prop values
- Emits interface for type-safe events
- Computed properties for reactive values
- Functions for event handlers

---

## Lazy Loading Pattern

### When to Lazy Load

Lazy load components that are:
- Heavy (drawers, modals, complex forms)
- Route-level components
- Modal/dialog content (not shown initially)
- Below-the-fold content
- Conditionally rendered components

### How to Lazy Load

```vue
<script setup lang="ts">
// Lazy load heavy component
const LazyUserDetailDrawer = defineAsyncComponent(() => 
  import('#user/components/user-management/user-detail/user-detail-drawer.vue')
)

const LazyHeavyComponent = defineAsyncComponent(() => 
  import('#feature/components/heavy-component.vue')
)

// With loading component
const LazyComponentWithLoading = defineAsyncComponent({
  loader: () => import('#feature/components/component.vue'),
  loadingComponent: () => import('#components/loading-spinner.vue'),
  delay: 200,
  timeout: 3000,
})
</script>

<template>
  <Suspense>
    <LazyUserDetailDrawer
      v-if="isDrawerVisible"
      :user-detail="userDetail"
      @closed="isDrawerVisible = false"
    />
  </Suspense>
</template>
```

**Example from user-list.vue:**

```vue
<script setup lang="ts">
defineOptions({
  name: 'UserList',
})

// Lazy load drawer components
const LazyUserDetailDrawer = defineAsyncComponent(() => 
  import('#user/components/user-management/user-detail/user-detail-drawer.vue')
)

const LazyGroupJobLocationPermissionsDrawer = defineAsyncComponent(() => 
  import('#user/components/shared/group-job-location-permission/group-job-location-permissions-drawer.vue')
)

const isShowUserDetailDrawer = ref(false)
const userDetail = ref<UserDetail>()

function showUserDetail(user: UserItem) {
  userDetail.value = user
  isShowUserDetailDrawer.value = true
}
</script>

<template>
  <div>
    <!-- User list content -->
    
    <LazyUserDetailDrawer
      v-if="isShowUserDetailDrawer"
      :user-detail="userDetail"
      @closed="isShowUserDetailDrawer = false"
    />
  </div>
</template>
```

---

## Loading States

### v-loading Directive

Use `v-loading` directive for loading states:

```vue
<script setup lang="ts">
const isLoading = ref(false)
const { data: users, pending } = useAsyncData('users', () => 
  useApiUsers().getUsers(),
  { server: false }
)
</script>

<template>
  <!-- Direct loading state -->
  <div v-loading="isLoading">
    <!-- Content -->
  </div>

  <!-- With useAsyncData pending -->
  <div v-loading="pending">
    <div v-for="user in users" :key="user.id">
      {{ user.name }}
    </div>
  </div>

  <!-- Button loading -->
  <ElButton
    :loading="isLoading"
    @click="handleAction"
  >
    Submit
  </ElButton>
</template>
```

### Suspense for Async Components

```vue
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

## Component Structure Template

### Recommended Order

```vue
<script setup lang="ts">
/**
 * Component description
 * What it does, when to use it
 */

// ============================================
// 1. IMPORTS
// ============================================
// Type imports first
import type { UserDetail, UserItem } from '#user/types/user'
import type { FormInstance } from 'element-plus'

// Component imports
import UserDetailSection from '#user/components/user-management/user-detail/user-detail-section/personal-section.vue'
import FormSectionCard from '#user/components/user-management/user-detail/form-section-card.vue'

// Composable imports
import { useApiUserContact } from '#user/composables'
import { useUserForm } from '#user/composables/useUserForm'
import { useFormatter } from '#common/composables'

// Constant imports
import { UserStatus, UserRole } from '#user/constants/user-management'

// ============================================
// 2. COMPONENT DEFINITION
// ============================================
defineOptions({
  name: 'UserDetailDrawer',
})

// ============================================
// 3. PROPS & EMITS
// ============================================
interface Props {
  /** User detail data */
  userDetail?: UserDetail
  /** Default role ID */
  defaultRoleId?: number
}

const props = withDefaults(defineProps<Props>(), {
  defaultRoleId: undefined,
})

interface Emits {
  (e: 'closed'): void
  (e: 'saved'): void
}

const emit = defineEmits<Emits>()

// ============================================
// 4. REFS & REACTIVE STATE
// ============================================
const isVisible = ref(true)
const isLoading = ref(false)
const formTemplate = useTemplateRef<InstanceType<typeof OlForm>>('userFormRef')
const userRoles = ref<RoleItem[]>([])
const serverErrors = ref<Record<string, number>>({})

// ============================================
// 5. COMPOSABLES
// ============================================
const { $gettext } = useLocale()
const { formatDate } = useFormatter()
const { loggedInUser } = useLoggedIn()
const { account: companySettings } = storeToRefs(useAccountStore())
const { getListSimpleRoles, createUser, updateUser } = useApiUserContact()
const { initFormData, formRules } = useUserForm(
  props.userDetail,
  props.defaultRoleId,
  isShowLoginEmail,
  serverErrors,
  getField
)

// ============================================
// 6. COMPUTED PROPERTIES
// ============================================
const isAddNewUser = computed(() => !props.userDetail)
const isDeactivatedUser = computed(() => 
  props.userDetail?.status === UserStatus.DEACTIVATED
)
const title = computed(() => 
  props.userDetail?.contact?.name || $gettext('Add New User')
)

// ============================================
// 7. METHODS
// ============================================
function getField(field: string | number) {
  return useGet(formData.value, field)
}

async function handleSave() {
  isLoading.value = true
  try {
    if (isAddNewUser.value) {
      await createUser(formData.value)
    }
    else {
      await updateUser(props.userDetail!.user_id, formData.value)
    }
    
    OlNotify.success({
      message: $gettext('User saved successfully'),
    })
    
    emit('saved')
    isVisible.value = false
  }
  catch (error) {
    OlNotify.error({
      message: $gettext('Failed to save user'),
    })
  }
  finally {
    isLoading.value = false
  }
}

// ============================================
// 8. LIFECYCLE HOOKS
// ============================================
onMounted(async () => {
  await loadInitialData()
})

onBeforeUnmount(() => {
  // Cleanup
})
</script>

<template>
  <OlDrawer
    v-model:value="isVisible"
    :title="title"
    :width="600"
  >
    <!-- Content -->
  </OlDrawer>
</template>
```

---

## Component Separation

### When to Split Components

**Split into multiple components when:**
- Component exceeds 300-400 lines
- Multiple distinct responsibilities
- Reusable sections
- Complex nested template logic
- Different data fetching requirements

**Example:**

```vue
<!-- ❌ AVOID - Monolithic -->
<script setup lang="ts">
// 500+ lines
// Search logic
// Filter logic
// List logic
// Action panel logic
// All in one component
</script>

<!-- ✅ PREFERRED - Modular -->
<!-- UserList.vue -->
<script setup lang="ts">
import UserSearch from './user-search.vue'
import UserFilter from './user-filter.vue'
import UserTable from './user-table.vue'
import UserActions from './user-actions.vue'
</script>

<template>
  <div>
    <UserSearch @search="handleSearch" />
    <UserFilter @filter="handleFilter" />
    <UserTable :users="filteredUsers" />
    <UserActions @action="handleAction" />
  </div>
</template>
```

### When to Keep Together

**Keep in same file when:**
- Component < 200-300 lines
- Tightly coupled logic
- Not reusable elsewhere
- Simple presentation component
- Related sub-components (use multiple `<script setup>` blocks if needed)

---

## Composables Pattern

### Business Logic Separation

Extract business logic into composables:

```typescript
// composables/useUserManagement.ts
export function useUserManagement() {
  const { $gettext } = useLocale()
  const { getListUsers, deactivateUser } = useApiUserContact()

  const {
    listData: users,
    listLoading,
    fetchData,
  } = useListData('users', {
    query: ['page', 'per_page', 'keyword'],
    defaultLimit: 20,
    server: false,
    async onFetch(query) {
      const response = await getListUsers({
        keyword: query.keyword,
        page: query.page,
        per_page: query.per_page,
      })
      return response.users
    },
  })

  async function handleDeactivate(userId: string) {
    try {
      await deactivateUser(userId)
      OlNotify.success({
        message: $gettext('User deactivated successfully'),
      })
      await fetchData()
    }
    catch (error) {
      OlNotify.error({
        message: $gettext('Failed to deactivate user'),
      })
    }
  }

  return {
    users,
    listLoading,
    fetchData,
    handleDeactivate,
  }
}
```

**Usage in component:**

```vue
<script setup lang="ts">
const {
  users,
  listLoading,
  fetchData,
  handleDeactivate,
} = useUserManagement()
</script>
```

---

## Template Refs Pattern

### useTemplateRef

Use `useTemplateRef` for type-safe template refs:

```vue
<script setup lang="ts">
import type { FormInstance } from 'element-plus'

// Type-safe template ref
const formRef = useTemplateRef<InstanceType<typeof OlForm>>('formRef')

// Access methods
async function validateForm() {
  await formRef.value?.validate()
}

function resetForm() {
  formRef.value?.resetFields()
}
</script>

<template>
  <OlForm ref="formRef">
    <!-- Form content -->
  </OlForm>
</template>
```

### Multiple Refs

```vue
<script setup lang="ts">
const formRef = useTemplateRef<InstanceType<typeof OlForm>>('formRef')
const drawerRef = useTemplateRef<InstanceType<typeof OlDrawer>>('drawerRef')
const selectRef = useTemplateRef<InstanceType<typeof OlSelectList>>('selectRef')
</script>
```

---

## Props and Emits Patterns

### Props with Defaults

```vue
<script setup lang="ts">
interface Props {
  /** User ID */
  userId: string
  /** Display mode */
  mode?: 'view' | 'edit'
  /** Show actions */
  showActions?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  mode: 'view',
  showActions: true,
})
</script>
```

### Emits with Type Safety

```vue
<script setup lang="ts">
interface Emits {
  (e: 'update', value: string): void
  (e: 'save', data: FormData): void
  (e: 'cancel'): void
  (e: 'delete', id: string): void
}

const emit = defineEmits<Emits>()

function handleSave() {
  emit('save', formData.value)
}
</script>
```

### Expose Pattern

```vue
<script setup lang="ts">
const isVisible = ref(false)

function open() {
  isVisible.value = true
}

function close() {
  isVisible.value = false
}

// Expose methods to parent
defineExpose({
  open,
  close,
  isVisible,
})
</script>
```

**Usage:**

```vue
<script setup lang="ts">
const drawerRef = useTemplateRef<InstanceType<typeof MyDrawer>>('drawerRef')

function showDrawer() {
  drawerRef.value?.open()
}
</script>
```

---

## Component Communication

### Props Down, Events Up

```vue
<!-- Parent -->
<script setup lang="ts">
const selectedId = ref<string | null>(null)
const users = ref<User[]>([])

function handleSelect(id: string) {
  selectedId.value = id
}
</script>

<template>
  <UserList
    :users="users"
    @select="handleSelect"
  />
</template>

<!-- Child -->
<script setup lang="ts">
interface Props {
  users: User[]
}

const props = defineProps<Props>()

interface Emits {
  (e: 'select', id: string): void
}

const emit = defineEmits<Emits>()

function handleClick(user: User) {
  emit('select', user.id)
}
</script>

<template>
  <div
    v-for="user in users"
    :key="user.id"
    @click="handleClick(user)"
  >
    {{ user.name }}
  </div>
</template>
```

### Provide/Inject for Deep Nesting

```vue
<!-- Parent -->
<script setup lang="ts">
const userData = ref<UserData | null>(null)

provide('userData', userData)
</script>

<!-- Deep Child -->
<script setup lang="ts">
const userData = inject<Ref<UserData | null>>('userData')
</script>
```

### Pinia Stores for Global State

```vue
<script setup lang="ts">
const { account } = storeToRefs(useAccountStore())

// Access reactive store state
const accountName = computed(() => account.value?.name)
</script>
```

---

## Advanced Patterns

### Dynamic Components

```vue
<script setup lang="ts">
import ComponentA from './component-a.vue'
import ComponentB from './component-b.vue'

const componentMap = {
  a: ComponentA,
  b: ComponentB,
}

const currentComponent = computed(() => 
  componentMap[props.type as keyof typeof componentMap]
)
</script>

<template>
  <component :is="currentComponent" />
</template>
```

### Slot Patterns

```vue
<!-- Parent Component -->
<script setup lang="ts">
defineSlots<{
  default(): any
  header(): any
  footer(): any
  item(props: { item: User }): any
}>()
</script>

<template>
  <div>
    <slot name="header" />
    <slot
      name="item"
      v-for="user in users"
      :key="user.id"
      :item="user"
    />
    <slot name="footer" />
  </div>
</template>

<!-- Usage -->
<MyComponent>
  <template #header>
    <h2>Users</h2>
  </template>
  
  <template #item="{ item }">
    <div>{{ item.name }}</div>
  </template>
  
  <template #footer>
    <ElButton>Add User</ElButton>
  </template>
</MyComponent>
```

### Conditional Rendering

```vue
<template>
  <!-- v-if for expensive components -->
  <LazyHeavyComponent v-if="shouldShow" />
  
  <!-- v-show for frequent toggles -->
  <div v-show="isVisible">
    Content
  </div>
  
  <!-- Multiple conditions -->
  <div v-if="pending">
    Loading...
  </div>
  <div v-else-if="error">
    Error: {{ error.message }}
  </div>
  <div v-else>
    {{ data }}
  </div>
</template>
```

---

## File Organization

### Component File Structure

```
areas/
  user/
    app/
      components/
        user-management/
          user-list.vue              # Main component
          user-list/
            user-search.vue          # Sub-component
            user-filter.vue          # Sub-component
            user-table.vue           # Sub-component
          user-detail/
            user-detail-drawer.vue   # Drawer component
            user-detail-section/
              personal-section.vue   # Section component
              permission-section.vue
      composables/
        useUserManagement.ts         # Business logic
        useUserForm.ts               # Form logic
        useApiUserContact.ts         # API calls
      types/
        user.ts                      # TypeScript types
      constants/
        user-management.ts           # Constants
```

### Import Aliases

```vue
<script setup lang="ts">
// Area imports (preferred)
import { useApiUsers } from '#user/composables'
import type { User } from '#user/types/user'

// Shared imports
import { useFormatter } from '#common/composables'
import PageCard from '#common/components/page/page-card.vue'

// Package imports
import { OlForm, OlDrawer } from '@paradoxai/ui'
</script>
```

---

## Performance Patterns

### Computed Properties

```vue
<script setup lang="ts">
const users = ref<User[]>([])
const searchQuery = ref('')

// Memoized computed
const filteredUsers = computed(() => {
  if (!searchQuery.value) return users.value
  
  return users.value.filter(user =>
    user.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})
</script>
```

### Watchers

```vue
<script setup lang="ts">
const searchQuery = ref('')
const results = ref([])

// Debounced watcher
watch(searchQuery, async (newQuery) => {
  if (newQuery.length > 2) {
    results.value = await api.search(newQuery)
  }
}, { debounce: 300 })

// Watch multiple sources
watch([userId, filters], async ([newUserId, newFilters]) => {
  await fetchData(newUserId, newFilters)
}, { immediate: true })
</script>
```

### Lazy Evaluation

```vue
<script setup lang="ts">
// Only compute when needed
const expensiveValue = computed(() => {
  if (!shouldCompute.value) return null
  return expensiveCalculation()
})
</script>
```

---

## Summary

**Modern Component Recipe:**
1. `<script setup lang="ts">` with TypeScript
2. `defineOptions({ name: 'ComponentName' })` for component name
3. `defineProps<Props>()` with TypeScript interfaces
4. `defineEmits<Emits>()` for type-safe events
5. Lazy load heavy components: `defineAsyncComponent()`
6. Use `v-loading` or `Suspense` for loading states
7. Extract business logic to composables
8. Use `useTemplateRef` for type-safe refs
9. Import aliases: `#area/composables`, `#common/components`
10. Computed properties for reactive values
11. `defineExpose` to expose methods to parent

**Component Structure Order:**
1. Imports (types, components, composables, constants)
2. `defineOptions`
3. Props & Emits
4. Refs & Reactive State
5. Composables
6. Computed Properties
7. Methods
8. Lifecycle Hooks

**See Also:**
- [data-fetching.md](data-fetching.md) - `useAsyncData` and `useListData` details
- [common-patterns.md](common-patterns.md) - Common UI patterns
- [complete-examples.md](complete-examples.md) - Full working examples
- [styling-guide.md](styling-guide.md) - UnoCSS styling patterns
