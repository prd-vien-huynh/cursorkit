# Routing Guide

Nuxt 3 file-based routing with lazy loading patterns and route metadata.

---

## Nuxt 3 Routing Overview

**Nuxt 3** uses file-based routing:
- Folder structure in `pages/` directory defines routes
- Lazy loading with `defineAsyncComponent`
- Type-safe routing with `useRoute()` and `useRouter()`
- Route metadata with `definePageMeta`
- Automatic code splitting

---

## Folder-Based Routing

### Directory Structure

```
pages/
  index.vue                    # Home route (/)
  about.vue                    # /about
  posts/
    index.vue                  # /posts
    create.vue                 # /posts/create
    [id].vue                   # /posts/:id (dynamic)
    [id]/
      edit.vue                 # /posts/:id/edit
  settings/
    [term]-roles/
      role/
        [id].vue               # /settings/:term-roles/role/:id
```

**Pattern**:
- `index.vue` = Route at that path
- `[param].vue` = Dynamic parameter (e.g., `[id].vue` = `:id`)
- Nested folders = Nested routes
- File name = Route path segment

---

## Basic Route Pattern

### Example from pages/posts/index.vue

```vue
<script setup lang="ts">
defineOptions({
  name: 'PostsIndex',
})

definePageMeta({
  auth: true,
  layout: 'cem',
  permission: {
    view: ACL_POSTS,
  },
})

useHead({
  title: 'Posts',
})

useBreadcrumb({
  hideCurrent: true,
  overrides: [
    {
      label: 'Posts',
    },
  ],
})
</script>

<template>
  <div class=":uno: p-4">
    <h1 class=":uno: text-2xl font-semibold mb-4">All Posts</h1>
    <PostsList />
  </div>
</template>
```

**Key Points:**
- `definePageMeta` for route configuration
- `useHead` for page metadata (title, etc.)
- `useBreadcrumb` for navigation breadcrumbs
- Layout specified in `definePageMeta`

---

## definePageMeta

### Basic Configuration

```vue
<script setup lang="ts">
definePageMeta({
  auth: true,              // Require authentication
  layout: 'cem',           // Use 'cem' layout
  permission: false,       // No permission check
})
</script>
```

### With Permission Check

```vue
<script setup lang="ts">
definePageMeta({
  auth: true,
  layout: 'cem',
  permission: {
    view: ACL_EVENT,        // Required permission key
    action: 'view',         // Permission action
  },
})
</script>
```

### With Middleware

```vue
<script setup lang="ts">
import { navigateTo } from '#imports'

definePageMeta({
  auth: true,
  layout: 'cem',
  middleware(to) {
    // Custom middleware logic
    if (!to.params.id) {
      return navigateTo({
        name: 'posts',
      })
    }
  },
})
</script>
```

### With Multiple Middleware

```vue
<script setup lang="ts">
definePageMeta({
  auth: true,
  layout: 'cem',
  middleware: [
    'remember-event-filter',  // Named middleware
    (to) => {
      // Inline middleware
      if (to.query.skip) {
        return navigateTo('/')
      }
    },
  ],
})
</script>
```

### With Custom Path

```vue
<script setup lang="ts">
definePageMeta({
  path: '/settings/:term-roles/role/:id',  // Custom route path
  auth: true,
  layout: 'cem',
})
</script>
```

### With Layout Options

```vue
<script setup lang="ts">
definePageMeta({
  layout: 'cem',
  onBack() {
    navigateTo('/example')
  },
  toolbarHideIcons: ['toolbar-icon-notification'],
})
</script>
```

---

## Dynamic Routes

### Single Parameter

```vue
<!-- pages/posts/[id].vue -->
<script setup lang="ts">
definePageMeta({
  auth: true,
  layout: 'cem',
})

const route = useRoute('posts-id')  // Type-safe route name
const postId = computed(() => route.params.id as string)
</script>

<template>
  <div>
    <h1>Post {{ postId }}</h1>
  </div>
</template>
```

### Multiple Parameters

```vue
<!-- pages/settings/[term]-roles/role/[id].vue -->
<script setup lang="ts">
definePageMeta({
  auth: true,
  layout: 'cem',
})

const route = useRoute('settings-term-roles-role-id')
const term = computed(() => route.params.term as string)
const roleId = computed(() => route.params.id as string)
</script>

<template>
  <div>
    <h1>Role {{ roleId }} for {{ term }}</h1>
  </div>
</template>
```

### Catch-All Routes

```vue
<!-- pages/docs/[...slug].vue -->
<script setup lang="ts">
const route = useRoute()
const slug = computed(() => (route.params.slug as string[]) || [])
</script>

<template>
  <div>
    <h1>Docs: {{ slug.join('/') }}</h1>
  </div>
</template>
```

---

## Navigation

### Programmatic Navigation

```vue
<script setup lang="ts">
import { navigateTo } from '#imports'

function handleClick() {
  navigateTo('/posts')
}

function handleNavigateWithQuery() {
  navigateTo({
    path: '/posts',
    query: { page: 1, filter: 'active' },
  })
}
</script>

<template>
  <button @click="handleClick">View Posts</button>
</template>
```

### With Route Name

```vue
<script setup lang="ts">
function handleNavigate() {
  navigateTo({
    name: 'posts-id',
    params: { id: '123' },
  })
}
</script>
```

### With Query Parameters

```vue
<script setup lang="ts">
function handleSearch() {
  navigateTo({
    name: 'posts',
    query: { 
      search: 'test',
      page: 1,
    },
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

function handlePush() {
  router.push({
    name: 'posts-id',
    params: { id: '123' },
  })
}
</script>
```

---

## Lazy Loading Components

### Using defineAsyncComponent

```vue
<script setup lang="ts">
const LazyPostsList = defineAsyncComponent(() => 
  import('#posts/components/posts-list.vue')
)

const LazyUserModal = defineAsyncComponent(() => 
  import('#user/components/user-modal.vue')
)
</script>

<template>
  <Suspense>
    <LazyPostsList />
  </Suspense>
</template>
```

### With Loading State

```vue
<script setup lang="ts">
const LazyHeavyComponent = defineAsyncComponent({
  loader: () => import('#features/components/heavy-component.vue'),
  loadingComponent: () => import('#components/loading-spinner.vue'),
  delay: 200,
  timeout: 3000,
})
</script>

<template>
  <LazyHeavyComponent />
</template>
```

### Why Lazy Load?

- Code splitting - smaller initial bundle
- Faster initial page load
- Load component code only when needed
- Better performance for large applications

---

## Route Metadata

### Page Head (SEO & Title)

```vue
<script setup lang="ts">
const { $gettext } = useLocale()

useHead({
  title: $gettext('Posts'),
  meta: [
    { name: 'description', content: 'List of all posts' },
  ],
})
</script>
```

### Breadcrumbs

```vue
<script setup lang="ts">
const { $gettext } = useLocale()

useBreadcrumb({
  hideCurrent: true,
  overrides: [
    {
      label: $gettext('Settings'),
      to: '/settings',
    },
    false,  // Skip a level
    {
      label: $gettext('Roles and Permissions'),
    },
  ],
})
</script>
```

### Dynamic Breadcrumbs

```vue
<script setup lang="ts">
const route = useRoute('posts-id')

useBreadcrumb({
  hideCurrent: true,
  append: () => [
    {
      label: `Post ${route.params.id}`,
    },
  ],
})
</script>
```

---

## Route Layouts

### Using Layouts

```vue
<script setup lang="ts">
definePageMeta({
  layout: 'cem',  // Use 'cem' layout
})
</script>

<template>
  <div>Page content</div>
</template>
```

### Available Layouts

- `cem` - Main CEM layout with sidebar
- `blank` - No layout (for auth pages, etc.)
- `page-column` - Column-based layout

### Nested Layouts

```vue
<script setup lang="ts">
definePageMeta({
  layout: 'cem',
})
</script>

<template>
  <NuxtLayout name="page-column">
    <Page>
      <PageCard title="Content">
        Page content here
      </PageCard>
    </Page>
  </NuxtLayout>
</template>
```

---

## Route Data Fetching

### Using useAsyncData

```vue
<script setup lang="ts">
definePageMeta({
  auth: true,
  layout: 'cem',
})

const route = useRoute('posts-id')

const { data: post, pending, error } = useAsyncData(
  `post-${route.params.id}`,
  () => $fetch(`/api/posts/${route.params.id}`)
)
</script>

<template>
  <div v-if="pending" class=":uno: p-4">Loading...</div>
  <div v-else-if="error" class=":uno: p-4 text-danger">Error loading post</div>
  <div v-else class=":uno: p-4">
    <h1>{{ post?.title }}</h1>
    <p>{{ post?.content }}</p>
  </div>
</template>
```

### With Refresh

```vue
<script setup lang="ts">
const { data, refresh } = useAsyncData('posts', () => 
  $fetch('/api/posts')
)

function handleRefresh() {
  refresh()
}
</script>
```

---

## Complete Route Example

```vue
<!-- pages/posts/[id].vue -->
<script setup lang="ts">
import { navigateTo } from '#imports'
import { ACL_POSTS } from '#common/constants'

defineOptions({
  name: 'PostDetail',
})

definePageMeta({
  auth: true,
  layout: 'cem',
  permission: {
    view: ACL_POSTS,
  },
  middleware(to) {
    if (!to.params.id || to.params.id === 'new') {
      return navigateTo({ name: 'posts' })
    }
  },
})

const { $gettext } = useLocale()
const route = useRoute('posts-id')
const router = useRouter()

useHead({
  title: computed(() => $gettext('Post') + ` ${route.params.id}`),
})

useBreadcrumb({
  hideCurrent: true,
  overrides: [
    {
      label: $gettext('Posts'),
      to: '/posts',
    },
    {
      label: computed(() => `Post ${route.params.id}`),
    },
  ],
})

const { data: post, pending } = useAsyncData(
  `post-${route.params.id}`,
  () => useApiPosts().getPost(route.params.id as string),
  {
    default: () => null,
  }
)

const LazyPostEditor = defineAsyncComponent(() => 
  import('#posts/components/post-editor.vue')
)

function handleSave() {
  // Save logic
  router.push({ name: 'posts' })
}
</script>

<template>
  <NuxtLayout name="page-column">
    <Page>
      <PageCard :title="$gettext('Post Detail')">
        <div v-if="pending" class=":uno: p-4">Loading...</div>
        <LazyPostEditor
          v-else-if="post"
          :post="post"
          @save="handleSave"
        />
      </PageCard>
    </NuxtLayout>
  </NuxtLayout>
</template>
```

---

## Route Guards & Middleware

### Global Middleware

Global middleware runs on every route:

```typescript
// middleware/1.auth.global.ts
export default defineNuxtRouteMiddleware((to) => {
  const { isAuthenticated } = useAuth()
  
  if (!isAuthenticated.value && to.meta.auth !== false) {
    return navigateTo('/login')
  }
})
```

### Named Middleware

```typescript
// middleware/remember-filter.ts
export default defineNuxtRouteMiddleware((to, from) => {
  // Remember filter state
  if (from.query.filter) {
    // Store filter state
  }
})
```

Use in page:
```vue
<script setup lang="ts">
definePageMeta({
  middleware: ['remember-filter'],
})
</script>
```

---

## Route Rules

### Redirects

```vue
<script setup lang="ts">
defineRouteRules({
  redirect: '/new-path',
})
</script>
```

### SSR Configuration

```vue
<script setup lang="ts">
defineRouteRules({
  ssr: false,  // Client-side only
  prerender: true,  // Pre-render at build time
})
</script>
```

---

## Type-Safe Routing

### Typed Route Names

```vue
<script setup lang="ts">
// Type-safe route name
const route = useRoute('posts-id')  // Autocomplete works
const router = useRouter()

// Type-safe navigation
router.push({
  name: 'posts-id',  // Autocomplete route names
  params: { id: '123' },
})
</script>
```

### Typed Route Params

```vue
<script setup lang="ts">
const route = useRoute('settings-term-roles-role-id')

// TypeScript knows these params exist
const term = route.params.term as string
const id = route.params.id as string
</script>
```

---

## Common Patterns

### Conditional Redirect

```vue
<script setup lang="ts">
definePageMeta({
  middleware(to) {
    const { isAdmin } = useAuthStore()
    
    if (!isAdmin.value) {
      return navigateTo('/')
    }
  },
})
</script>
```

### Query Parameter Handling

```vue
<script setup lang="ts">
const route = useRoute()
const router = useRouter()

const searchQuery = computed({
  get: () => (route.query.q as string) || '',
  set: (value) => {
    router.replace({
      query: { ...route.query, q: value },
    })
  },
})
</script>
```

### Route Change Detection

```vue
<script setup lang="ts">
const route = useRoute()

watch(() => route.params.id, (newId, oldId) => {
  if (newId !== oldId) {
    // Handle route param change
    refreshData()
  }
})
</script>
```

---

## Summary

**Routing Checklist:**
- ✅ File-based: `pages/my-route/index.vue` or `pages/my-route.vue`
- ✅ Use `definePageMeta` for route configuration
- ✅ Lazy load heavy components: `defineAsyncComponent()`
- ✅ Use `useRoute()` for accessing route data
- ✅ Use `navigateTo()` for programmatic navigation
- ✅ Use `useHead()` for page metadata
- ✅ Use `useBreadcrumb()` for breadcrumbs
- ✅ Dynamic routes: `[param].vue` for `:param`
- ✅ Type-safe route names with `useRoute('route-name')`

**Route Configuration Options:**
- `auth: boolean` - Require authentication
- `layout: string` - Layout name ('cem', 'blank', etc.)
- `permission: object` - Permission requirements
- `middleware: function | array` - Route middleware
- `path: string` - Custom route path

**See Also:**
- [component-patterns.md](component-patterns.md) - Component structure
- [styling-guide.md](styling-guide.md) - Styling with UnoCSS
- [complete-examples.md](complete-examples.md) - Full route examples
