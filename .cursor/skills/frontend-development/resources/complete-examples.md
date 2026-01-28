# Complete Examples

Full working examples combining all modern patterns: Vue 3 Composition API, lazy loading, `useAsyncData`, styling with UnoCSS, routing, and error handling.

---

## Example 1: Complete Modern Component

Combines: Vue 3 Composition API, `useAsyncData`, computed properties, styling, error handling

```vue
<script setup lang="ts">
import { z } from 'zod'
import type { FormInstance } from 'element-plus'
import { useApiUsers } from '#user/composables'

const props = defineProps<{
  userId: string
  onUpdate?: () => void
}>()

const emit = defineEmits<{
  update: []
}>()

const formRef = ref<FormInstance>()
const isEditing = ref(false)
const isLoading = ref(false)

const {
  data: user,
  error: userError,
  pending: userPending,
  refresh: refreshUser,
} = useAsyncData(
  `user-${props.userId}`,
  () => useApiUsers().getUser(props.userId),
  {
    server: false,
    watch: [() => props.userId],
  }
)

whenError(userError, () => {
  OlNotify.error({
    message: 'Failed to load user',
  })
})

const userSchema = z.object({
  firstName: z.string().min(1, 'First name is required'),
  lastName: z.string().min(1, 'Last name is required'),
  email: z.string().email('Invalid email address'),
})

type UserFormData = z.infer<typeof userSchema>

const formModel = reactive<UserFormData>({
  firstName: '',
  lastName: '',
  email: '',
})

// Initialize form when user loads
watch(user, (newUser) => {
  if (newUser) {
    formModel.firstName = newUser.first_name || ''
    formModel.lastName = newUser.last_name || ''
    formModel.email = newUser.email || ''
  }
}, { immediate: true })

// Computed full name
const fullName = computed(() => {
  if (!user.value) return ''
  return `${user.value.first_name} ${user.value.last_name}`
})

async function handleEdit() {
  isEditing.value = true
}

async function handleSave() {
  if (!formRef.value) return

  await formRef.value.validate(async (isValid) => {
    if (isValid) {
      isLoading.value = true
      try {
        const data = userSchema.parse(formModel)
        await useApiUsers().updateUser(props.userId, data)
        
        await refreshUser()
        OlNotify.success({
          message: 'Profile updated successfully',
        })
        isEditing.value = false
        emit('update')
      }
      catch (error) {
        OlNotify.error({
          message: 'Failed to update profile',
        })
      }
      finally {
        isLoading.value = false
      }
    }
  })
}

function handleCancel() {
  isEditing.value = false
  // Reset form to original values
  if (user.value) {
    formModel.firstName = user.value.first_name || ''
    formModel.lastName = user.value.last_name || ''
    formModel.email = user.value.email || ''
  }
}
</script>

<template>
  <PageCard
    v-loading="userPending"
    class=":uno: max-w-2xl mx-auto"
  >
    <div class=":uno: flex items-center gap-4 mb-6">
      <div class=":uno: w-16 h-16 rounded-full bg-primary-100 flex items-center justify-center text-primary text-2xl font-semibold">
        {{ user?.first_name?.[0] }}{{ user?.last_name?.[0] }}
      </div>
      <div>
        <h2 class=":uno: text-2xl font-semibold text-grey-900">
          {{ fullName }}
        </h2>
        <p class=":uno: text-grey-600">{{ user?.email }}</p>
      </div>
    </div>

    <OlForm
      ref="formRef"
      :model="formModel"
      :schema="userSchema"
    >
      <OlFormItem
        label="First Name"
        prop="firstName"
      >
        <ElInput
          v-model="formModel.firstName"
          :disabled="!isEditing"
        />
      </OlFormItem>

      <OlFormItem
        label="Last Name"
        prop="lastName"
      >
        <ElInput
          v-model="formModel.lastName"
          :disabled="!isEditing"
        />
      </OlFormItem>

      <OlFormItem
        label="Email"
        prop="email"
      >
        <ElInput
          v-model="formModel.email"
          type="email"
          :disabled="!isEditing"
        />
      </OlFormItem>

      <div class=":uno: flex gap-2 mt-4">
        <ElButton
          v-if="!isEditing"
          type="primary"
          @click="handleEdit"
        >
          Edit Profile
        </ElButton>
        <template v-else>
          <ElButton
            type="primary"
            :loading="isLoading"
            @click="handleSave"
          >
            Save
          </ElButton>
          <ElButton @click="handleCancel">
            Cancel
          </ElButton>
        </template>
      </div>
    </OlForm>
  </PageCard>
</template>
```

---

## Example 2: Complete Route with Lazy Loading

```vue
<!-- pages/users/[id].vue -->
<script setup lang="ts">
import { navigateTo } from '#imports'
import { ACL_USERS } from '#common/constants'

defineOptions({
  name: 'UserDetail',
})

definePageMeta({
  auth: true,
  layout: 'cem',
  permission: {
    view: ACL_USERS,
  },
  middleware(to) {
    if (!to.params.id) {
      return navigateTo({ name: 'users' })
    }
  },
})

const { $gettext } = useLocale()
const route = useRoute('users-id')

useHead({
  title: computed(() => $gettext('User') + ` ${route.params.id}`),
})

useBreadcrumb({
  hideCurrent: true,
  overrides: [
    {
      label: $gettext('Users'),
      to: '/users',
    },
    {
      label: computed(() => `User ${route.params.id}`),
    },
  ],
})

// Lazy load heavy component
const LazyUserProfile = defineAsyncComponent(() => 
  import('#user/components/user-profile.vue')
)
</script>

<template>
  <NuxtLayout name="page-column">
    <Page>
      <PageCard :title="$gettext('User Profile')">
        <Suspense>
          <LazyUserProfile
            :user-id="route.params.id as string"
            @update="refreshUser"
          />
        </Suspense>
      </PageCard>
    </NuxtLayout>
  </NuxtLayout>
</template>
```

---

## Example 3: Form with Validation and Select

```vue
<script setup lang="ts">
import { z } from 'zod'
import type { FormInstance } from 'element-plus'

const formRef = ref<FormInstance>()

const userSchema = z.object({
  username: z.string().min(3, 'Username must be at least 3 characters'),
  email: z.string().email('Invalid email address'),
  role: z.string().min(1, 'Role is required'),
  department: z.string().optional(),
})

type FormData = z.infer<typeof userSchema>

const formModel = reactive<FormData>({
  username: '',
  email: '',
  role: '',
  department: '',
})

const {
  data: roles,
  pending: rolesPending,
} = useAsyncData('roles', () => useApiRoles().getRoles(), {
  server: false,
})

const {
  data: departments,
  pending: departmentsPending,
} = useAsyncData('departments', () => useApiDepartments().getDepartments(), {
  server: false,
})

async function handleSubmit() {
  if (!formRef.value) return

  await formRef.value.validate(async (isValid) => {
    if (isValid) {
      try {
        const data = userSchema.parse(formModel)
        await useApiUsers().createUser(data)
        
        OlNotify.success({
          message: 'User created successfully',
        })
        
        // Reset form
        formModel.username = ''
        formModel.email = ''
        formModel.role = ''
        formModel.department = ''
        formRef.value.resetFields()
      }
      catch (error) {
        OlNotify.error({
          message: 'Failed to create user',
        })
      }
    }
  })
}
</script>

<template>
  <PageCard title="Create User">
    <OlForm
      ref="formRef"
      :model="formModel"
      :schema="userSchema"
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
        label="Role"
        prop="role"
      >
        <OlSelectList
          v-model="formModel.role"
          :options="roles || []"
          :loading="rolesPending"
          :label-reducer="(opt: Role) => opt.name"
          :value-reducer="(opt: Role) => opt.id"
          placeholder="Select role"
        />
      </OlFormItem>

      <OlFormItem
        label="Department"
        prop="department"
      >
        <OlSelectList
          v-model="formModel.department"
          :options="departments || []"
          :loading="departmentsPending"
          :label-reducer="(opt: Department) => opt.name"
          :value-reducer="(opt: Department) => opt.id"
          placeholder="Select department"
          clearable
        />
      </OlFormItem>

      <ElButton
        type="primary"
        @click="handleSubmit"
      >
        Create User
      </ElButton>
    </OlForm>
  </PageCard>
</template>
```

---

## Example 4: List with Search and Pagination

```vue
<script setup lang="ts">
import { useApiUsers } from '#user/composables'

const {
  fetchMoreData: fetchMoreUsers,
  fetchData: fetchUsers,
  listData: users,
  listQuery,
  listLoading,
  canLoadMore,
} = useListData('users-list', {
  query: ['page', 'per_page', 'keyword'],
  defaultLimit: 20,
  server: false,
  lazy: true,
  immediate: false,
  async onFetch(query) {
    const response = await useApiUsers().list({
      keyword: query.keyword,
      page: query.page,
      per_page: query.per_page,
    })
    return response.users
  },
})

onMounted(() => {
  fetchUsers()
})
</script>

<template>
  <PageCard title="Users">
    <div class=":uno: mb-4">
      <ElInput
        v-model="listQuery.keyword"
        placeholder="Search users..."
        clearable
      >
        <template #prefix>
          <i class=":uno: i-ol-search icon:text-grey-400" />
        </template>
      </ElInput>
    </div>

    <div
      v-loading="listLoading"
      class=":uno: space-y-2"
    >
      <div
        v-for="user in users"
        :key="user.id"
        class=":uno: p-4 border border-grey-200 rounded hover:bg-grey-50"
      >
        <div class=":uno: flex items-center justify-between">
          <div>
            <h3 class=":uno: font-semibold text-grey-900">
              {{ user.first_name }} {{ user.last_name }}
            </h3>
            <p class=":uno: text-sm text-grey-600">{{ user.email }}</p>
          </div>
        </div>
      </div>

      <ElButton
        v-if="canLoadMore"
        :loading="listLoading"
        class=":uno: w-full mt-4"
        @click="fetchMoreUsers"
      >
        Load More
      </ElButton>
    </div>
  </PageCard>
</template>
```

---

## Example 5: Drawer with Form

```vue
<script setup lang="ts">
import { z } from 'zod'
import type { FormInstance } from 'element-plus'

const props = defineProps<{
  userId?: string
}>()

const emit = defineEmits<{
  close: []
  saved: []
}>()

const isDrawerVisible = ref(false)
const formRef = ref<FormInstance>()
const isLoading = ref(false)

const userSchema = z.object({
  name: z.string().min(1, 'Name is required'),
  email: z.string().email('Invalid email address'),
})

type FormData = z.infer<typeof userSchema>

const formModel = reactive<FormData>({
  name: '',
  email: '',
})

const {
  data: user,
  refresh: refreshUser,
} = useAsyncData(
  `user-${props.userId}`,
  () => props.userId ? useApiUsers().getUser(props.userId) : null,
  {
    server: false,
    immediate: !!props.userId,
  }
)

watch(user, (newUser) => {
  if (newUser) {
    formModel.name = `${newUser.first_name} ${newUser.last_name}`
    formModel.email = newUser.email
  }
}, { immediate: true })

function openDrawer() {
  isDrawerVisible.value = true
}

function closeDrawer() {
  isDrawerVisible.value = false
  emit('close')
}

async function handleSave() {
  if (!formRef.value) return

  await formRef.value.validate(async (isValid) => {
    if (isValid) {
      isLoading.value = true
      try {
        const data = userSchema.parse(formModel)
        
        if (props.userId) {
          await useApiUsers().updateUser(props.userId, data)
        }
        else {
          await useApiUsers().createUser(data)
        }
        
        OlNotify.success({
          message: props.userId ? 'User updated successfully' : 'User created successfully',
        })
        
        await refreshUser()
        closeDrawer()
        emit('saved')
      }
      catch (error) {
        OlNotify.error({
          message: 'Failed to save user',
        })
      }
      finally {
        isLoading.value = false
      }
    }
  })
}

defineExpose({
  open: openDrawer,
})
</script>

<template>
  <OlDrawer
    v-model:value="isDrawerVisible"
    :title="userId ? 'Edit User' : 'Create User'"
    :width="600"
  >
    <template #header>
      <div class=":uno: flex items-center gap-2">
        <i class=":uno: i-ol-user icon:text-primary" />
        <span>{{ userId ? 'Edit User' : 'Create User' }}</span>
      </div>
    </template>

    <div class=":uno: p-4">
      <OlForm
        ref="formRef"
        :model="formModel"
        :schema="userSchema"
      >
        <OlFormItem
          label="Name"
          prop="name"
        >
          <ElInput v-model="formModel.name" />
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
      </OlForm>
    </div>

    <template #footer>
      <div class=":uno: flex justify-end gap-2">
        <ElButton @click="closeDrawer">
          Cancel
        </ElButton>
        <ElButton
          type="primary"
          :loading="isLoading"
          @click="handleSave"
        >
          Save
        </ElButton>
      </div>
    </template>
  </OlDrawer>
</template>
```

---

## Example 6: Parallel Data Fetching

```vue
<script setup lang="ts">
const route = useRoute('dashboard')

// Fetch all data in parallel
const { data: stats } = useAsyncData('dashboard-stats', () => 
  useApiStats().getStats(),
  { server: false }
)

const { data: users } = useAsyncData('dashboard-users', () => 
  useApiUsers().getActiveUsers(),
  { server: false }
)

const { data: activity } = useAsyncData('dashboard-activity', () => 
  useApiActivity().getRecent(),
  { server: false }
)
</script>

<template>
  <div class=":uno: p-4">
    <div class=":uno: grid grid-cols-1 md:grid-cols-3 gap-4">
      <PageCard title="Stats">
        <div v-if="stats">
          <p class=":uno: text-2xl font-semibold">{{ stats.total }}</p>
        </div>
      </PageCard>

      <PageCard title="Active Users">
        <div v-if="users">
          <p class=":uno: text-2xl font-semibold">{{ users.length }}</p>
        </div>
      </PageCard>

      <PageCard title="Recent Activity">
        <div v-if="activity">
          <p class=":uno: text-2xl font-semibold">{{ activity.length }}</p>
        </div>
      </PageCard>
    </div>
  </div>
</template>
```

---

## Example 7: SelectList with Lazy Loading and Search

```vue
<script setup lang="ts">
const selectedSurvey = ref<string | null>(null)
const searchKeyword = ref('')

const {
  fetchMoreData,
  listData: surveys,
  listQuery,
  listLoading,
  canLoadMore,
} = useListData('surveys-list', {
  query: ['page', 'per_page', 'keyword'],
  defaultLimit: 20,
  server: false,
  lazy: true,
  immediate: false,
  async onFetch(query) {
    const response = await useApiSurveys().list({
      keyword: query.keyword,
      page: query.page,
      per_page: query.per_page,
    })
    return response.surveys
  },
})

onMounted(() => {
  // Initial fetch
  listQuery.value.keyword = ''
  fetchData()
})

// Watch search keyword and refetch
watch(searchKeyword, (newKeyword) => {
  listQuery.value.keyword = newKeyword
  fetchData()
}, { debounce: 300 })

function handleSelect(survey: Survey) {
  selectedSurvey.value = survey.id
  OlNotify.success({
    message: `Selected: ${survey.name}`,
  })
}
</script>

<template>
  <OlFormItem
    label="Survey"
    prop="survey"
  >
    <OlSelectList
      v-model="selectedSurvey"
      v-model:search="searchKeyword"
      :options="surveys"
      :loading="listLoading"
      :can-load-more="canLoadMore"
      filterable
      lazy
      size="large"
      :label-reducer="(item: Survey) => item.name"
      :value-reducer="(item: Survey) => item.id"
      :placeholder="$gettext('Select survey')"
      :search-placeholder="$gettext('Search survey')"
      :no-match-text="$gettext('No results found')"
      @loadmore="fetchMoreData"
      @select="handleSelect"
    >
      <template #item="{ item }">
        <span class=":uno: block truncate">{{ item.name }}</span>
      </template>
      
      <template #empty>
        <div
          v-if="listLoading"
          class=":uno: flex justify-center p-4"
        >
          <OlSpinner />
        </div>
        <div
          v-else
          class=":uno: p-4 text-center text-grey-400"
        >
          {{ $gettext('No surveys found') }}
        </div>
      </template>
    </OlSelectList>
  </OlFormItem>
</template>
```

---

## When to Use OlSelectList vs OlDropdown

### Use OlSelectList When:

1. **Form Input Selection**
   - User needs to select a value for a form field
   - Selected value should be displayed in an input field
   - Value needs to be bound to `v-model`

```vue
<OlFormItem label="Role" prop="role">
  <OlSelectList
    v-model="formModel.role"
    :options="roles"
    :label-reducer="(opt) => opt.name"
    :value-reducer="(opt) => opt.id"
    placeholder="Select role"
  />
</OlFormItem>
```

2. **Search/Filter Required**
   - Need to filter through many options
   - Users need to search for specific items

```vue
<OlSelectList
  v-model="selectedItem"
  v-model:search="searchKeyword"
  :options="items"
  filterable
  :filter-method="handleFilter"
  placeholder="Search and select..."
/>
```

3. **Lazy Loading or Infinite Scroll**
   - Large datasets that need pagination
   - Loading more items on scroll

```vue
<OlSelectList
  v-model="selectedItem"
  :options="items"
  :can-load-more="canLoadMore"
  :loading="loading"
  lazy
  @loadmore="fetchMoreData"
/>
```

4. **Mobile Bottomsheet Mode**
   - Better UX on mobile devices
   - Needs confirmation mode

```vue
<OlSelectList
  v-model="selectedItem"
  :options="items"
  :bottomsheet="isMobile"
  :confirmation="isMobile"
  :with-footer="isMobile"
/>
```

5. **Virtual Scrolling**
   - Very large lists (1000+ items)
   - Performance optimization needed

```vue
<OlSelectList
  v-model="selectedItem"
  :options="largeList"
  virtual
  :virtual-options="{
    itemSize: 40,
    buffer: 5,
  }"
/>
```

6. **Custom Label/Value Reducers**
   - Complex data structures
   - Need custom display logic

```vue
<OlSelectList
  v-model="selectedItem"
  :options="complexItems"
  :label-reducer="(item) => `${item.firstName} ${item.lastName}`"
  :value-reducer="(item) => item.id"
  :selected-label-reducer="(selected) => selected?.fullName || 'Select'"
/>
```

### Use OlDropdown When:

1. **Action Menu**
   - Menu items that trigger actions (Edit, Delete, View)
   - Not selecting a value, but performing actions

```vue
<OlDropdown
  placement="bottom-end"
  trigger="click"
>
  <ElButton>
    <i class=":uno: i-ol-ellipsis-vertical" />
  </ElButton>
  
  <template #dropdown>
    <OlDropdownMenu>
      <ElDropdownItem @click="handleEdit">
        <i class=":uno: i-ol-pencil icon:mr-3" />
        Edit
      </ElDropdownItem>
      <ElDropdownItem @click="handleDelete">
        <i class=":uno: i-ol-trash icon:mr-3" />
        Delete
      </ElDropdownItem>
    </OlDropdownMenu>
  </template>
</OlDropdown>
```

2. **Simple Menu Items**
   - No search/filter needed
   - Small, fixed list of options
   - Actions, not value selection

```vue
<OlDropdown>
  <ElButton>Options</ElButton>
  <template #dropdown>
    <OlDropdownMenu>
      <ElDropdownItem command="option1">Option 1</ElDropdownItem>
      <ElDropdownItem command="option2">Option 2</ElDropdownItem>
      <ElDropdownItem command="option3">Option 3</ElDropdownItem>
    </OlDropdownMenu>
  </template>
</OlDropdown>
```

3. **No Selected Value Display**
   - Don't need to show selected value in input
   - Just need a menu that opens on click

```vue
<OlDropdown>
  <ElButton type="primary">
    Actions
    <i class=":uno: i-ol-chevron-down ml-2" />
  </ElButton>
  <template #dropdown>
    <OlDropdownMenu>
      <ElDropdownItem @click="handleAction1">Action 1</ElDropdownItem>
      <ElDropdownItem @click="handleAction2">Action 2</ElDropdownItem>
    </OlDropdownMenu>
  </template>
</OlDropdown>
```

4. **Context Menu**
   - Right-click menu
   - Quick actions

```vue
<OlDropdown
  trigger="contextmenu"
>
  <div>Right-click me</div>
  <template #dropdown>
    <OlDropdownMenu>
      <ElDropdownItem>Copy</ElDropdownItem>
      <ElDropdownItem>Paste</ElDropdownItem>
    </OlDropdownMenu>
  </template>
</OlDropdown>
```

### Comparison Table

| Feature | OlSelectList | OlDropdown |
|---------|--------------|------------|
| **Use Case** | Form input selection | Action menu |
| **v-model** | ✅ Yes | ❌ No |
| **Search/Filter** | ✅ Yes | ❌ No |
| **Lazy Loading** | ✅ Yes | ❌ No |
| **Bottomsheet** | ✅ Yes | ✅ Yes |
| **Virtual Scroll** | ✅ Yes | ❌ No |
| **Selected Value Display** | ✅ Yes | ❌ No |
| **Confirmation Mode** | ✅ Yes | ❌ No |
| **Custom Reducers** | ✅ Yes | ❌ No |
| **Action Items** | ❌ No | ✅ Yes |
| **Simple Menu** | ❌ No | ✅ Yes |

### Quick Decision Guide

**Choose OlSelectList if:**
- ✅ Selecting a value for a form field
- ✅ Need search/filter functionality
- ✅ Large dataset with pagination
- ✅ Mobile bottomsheet needed
- ✅ Need to display selected value

**Choose OlDropdown if:**
- ✅ Action menu (Edit, Delete, View)
- ✅ Simple fixed list of options
- ✅ No value selection needed
- ✅ Context menu
- ✅ Quick actions

---

## Example 8: Complete Feature Structure

Real example based on `areas/user/`:

```
areas/
  user/
    app/
      pages/
        settings/
          [term]-roles/
            role/
              [id].vue          # Route component
      components/
        user-management/
          user-list.vue         # List component
          user-detail/
            user-detail-drawer.vue  # Drawer component
        role-management/
          role-list.vue
      composables/
        useApiUserRole.ts       # API composable
        useUserForm.ts          # Form logic
      constants/
        role-permission.ts      # Constants
      utils/
        rolePermission.ts       # Utility functions
```

### API Composable (useApiUserRole.ts)

```typescript
export function useApiUserRole() {
  const { $http } = useNuxtApp()

  return {
    getUserRoleDetail: async (roleId: string) => {
      return await $http.get(`/api/user-roles/${roleId}`)
    },

    getListUserRoles: async () => {
      return await $http.get('/api/user-roles')
    },

    createUserRole: async (data: CreateRolePayload) => {
      return await $http.post('/api/user-roles', data)
    },

    updateUserRole: async (roleId: string, data: UpdateRolePayload) => {
      return await $http.put(`/api/user-roles/${roleId}`, data)
    },
  }
}
```

### Types (types/user-role.ts)

```typescript
export interface UserRole {
  id: string
  name: string
  is_default: boolean
  permissions: Permission[]
  created_at: string
  updated_at: string
}

export interface CreateRolePayload {
  name: string
  permissions: string[]
}

export type UpdateRolePayload = Partial<CreateRolePayload>
```

---

## Example 9: Optimistic Update Pattern

```vue
<script setup lang="ts">
const { data: users, refresh: refreshUsers } = useAsyncData('users', () => 
  useApiUsers().getUsers(),
  { server: false }
)

const isLoading = ref(false)

async function handleToggleStatus(userId: string) {
  // Optimistic update
  if (users.value) {
    const user = users.value.find(u => u.id === userId)
    if (user) {
      user.active = !user.active
    }
  }

  isLoading.value = true
  try {
    await useApiUsers().toggleStatus(userId)
    OlNotify.success({
      message: 'Status updated',
    })
  }
  catch (error) {
    // Rollback on error
    await refreshUsers()
    OlNotify.error({
      message: 'Failed to update status',
    })
  }
  finally {
    isLoading.value = false
  }
}
</script>
```

---

## Example 10: SelectList with Confirmation Mode

```vue
<script setup lang="ts">
const selectedTimezone = ref<string | null>(null)

const {
  data: timezones,
  pending: timezonesPending,
} = useAsyncData('timezones', () => 
  useApiTimezones().getTimezones(),
  { server: false }
)

const isMobile = computed(() => useBreakpoints().smaller('sm'))
</script>

<template>
  <OlSelectList
    v-model="selectedTimezone"
    :options="timezones || []"
    :loading="timezonesPending"
    :bottomsheet="isMobile"
    :confirmation="isMobile"
    :with-footer="isMobile"
    :label-reducer="(opt: Timezone) => opt.name"
    :value-reducer="(opt: Timezone) => opt.id"
    placeholder="Select timezone"
    size="large"
  >
    <template
      v-if="isMobile"
      #prefix-header
    >
      <div class=":uno: px-8 py-4 font-semibold text-[16px] border-b border-grey-400">
        {{ $gettext('Timezone') }}
      </div>
    </template>
  </OlSelectList>
</template>
```

---

## Example 11: Dropdown Action Menu

```vue
<script setup lang="ts">
const props = defineProps<{
  userId: string
  onEdit?: (id: string) => void
  onDelete?: (id: string) => void
}>()

function handleEdit() {
  props.onEdit?.(props.userId)
}

async function handleDelete() {
  try {
    await OlDialogService({
      type: 'danger',
      title: 'Confirm Delete',
      body: 'Are you sure you want to delete this user?',
      textConfirm: 'Delete',
      textCancel: 'Cancel',
    })

    await useApiUsers().deleteUser(props.userId)
    OlNotify.success({
      message: 'User deleted successfully',
    })
    props.onDelete?.(props.userId)
  }
  catch {
    // User cancelled
  }
}
</script>

<template>
  <OlDropdown
    placement="bottom-end"
    trigger="click"
    :hide-on-click="true"
  >
    <ElButton
      class=":uno: border-0 rounded-full text-grey hover:text-grey-500 hover:bg-grey-200"
      size="small"
    >
      <i class=":uno: i-ol-ellipsis-vertical" />
    </ElButton>
    
    <template #dropdown>
      <OlDropdownMenu class=":uno: w-40">
        <ElDropdownItem
          class=":uno: h-11"
          @click="handleEdit"
        >
          <i class=":uno: i-ol-pencil icon:mr-3 icon:h-6 icon:w-6 icon:text-grey" />
          {{ $gettext('Edit') }}
        </ElDropdownItem>
        <ElDropdownItem
          class=":uno: h-11"
          @click="handleDelete"
        >
          <i class=":uno: i-ol-trash icon:mr-3 icon:h-6 icon:w-6 icon:text-grey" />
          {{ $gettext('Delete') }}
        </ElDropdownItem>
      </OlDropdownMenu>
    </template>
  </OlDropdown>
</template>
```

---

## Summary

**Key Takeaways:**

1. **Component Pattern**: Vue 3 Composition API + `defineAsyncComponent` + `useAsyncData`
2. **Feature Structure**: Organized by area/feature with pages, components, composables
3. **Routing**: Nuxt 3 file-based routing with `definePageMeta`
4. **Data Fetching**: `useAsyncData` for single fetches, `useListData` for paginated lists
5. **Forms**: `OlForm` with Zod schema validation
6. **Error Handling**: `OlNotify` + `whenError` composable
7. **Performance**: `computed`, `watch`, lazy loading, virtual scrolling
8. **Styling**: UnoCSS with `:uno:` prefix
9. **SelectList vs Dropdown**: Use SelectList for form inputs, Dropdown for action menus

**When to Use OlSelectList:**
- Form input selection
- Search/filter needed
- Lazy loading/pagination
- Mobile bottomsheet
- Display selected value

**When to Use OlDropdown:**
- Action menus (Edit, Delete, View)
- Simple fixed options
- No value selection
- Context menus

**See other resources for detailed explanations of each pattern.**
