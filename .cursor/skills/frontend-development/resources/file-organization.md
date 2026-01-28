# File Organization

Proper file and directory structure for maintainable, scalable frontend code in Nuxt 3 areas-based architecture.

---

## Areas Architecture

### Overview

The project uses a **monorepo structure** with **areas** - each area is a Nuxt 3 layer/module that can extend other areas. This allows for:

- **Modular development**: Each area is self-contained
- **Code sharing**: Areas can extend shared areas (e.g., `*-shared`)
- **Independent deployment**: Areas can be built and deployed separately
- **Clear boundaries**: Domain-specific code is isolated

### Area Structure

Each area follows this structure:

```
areas/
  my-area/
    app/                    # Application code
      api/                  # API services
      components/           # Vue components
      composables/         # Composables/hooks
      layouts/             # Nuxt layouts
      pages/               # Nuxt pages/routes
      stores/              # Pinia stores
      utils/               # Utility functions
      tests/               # Test files
    constants/             # Constants
    schemas/               # Zod/validation schemas
    types/                 # TypeScript types
    nuxt.config.ts         # Nuxt configuration
    package.json           # Area dependencies
    tsconfig.json          # TypeScript config
```

---

## Area Directory Structure (Detailed)

### Complete Area Example

Based on `areas/site-studio/` structure:

```
areas/
  site-studio/
    app/
      api/
        _composables_/
          index.ts          # Composable wrappers (useApiX)
        site-components.ts  # API service
        site-details.ts
        site-elements.ts
        sites.ts

      components/
        common/             # Shared components within area
          base-card.vue
          loading-indicator.vue
        dashboard/          # Feature-specific components
          dashboard-body/
            index.vue
          dashboard-header/
            index.vue
        list-site/
          site-data-table.vue
        site-details/
          base-layout/
          components/
          explorer/
          media/
          settings/

      composables/
        useDashboard.ts
        useSiteDetailsActivity.ts
        useVariableLibrary.ts

      layouts/
        cem-site-studio.vue
        cem-variable-library.vue

      pages/
        site-studio/
          [id]/
            index.vue
          index.vue
        site-studio.vue

      stores/
        explorer.ts
        site-studio.ts

      tests/
        __mocks__/
        components/
        composables/
        fixtures/
        helpers/
        stores/
        utils/

      utils/
        breadcrumb.ts
        code-editor.ts
        file-tree.ts

    constants/
      sites.ts
      site-details.ts
      site-elements.ts

    schemas/
      sites.ts
      site-settings.ts

    types/
      sites.d.ts
      site-details.d.ts
      site-elements.d.ts

    nuxt.config.ts
    package.json
    tsconfig.json
```

---

## Subdirectory Guidelines

### app/api/ Directory

**Purpose**: Centralized API calls for the area

**Structure:**
```
app/api/
  _composables_/
    index.ts              # Export all useApiX composables
  myFeatureApi.ts         # API service (default export)
  anotherApi.ts
```

**Pattern:**
```typescript
// areas/my-area/app/api/myFeatureApi.ts
export default function () {
  async function getEntity(id: string) {
    return await $http.$get<{ data: MyEntity }>(`/my-feature/${id}`)
  }

  async function list(params: ListParams) {
    return await $http.$get<{ rows: MyEntity[] }>('/my-feature', { params })
  }

  async function create(payload: CreatePayload) {
    return await $http.$post<{ data: MyEntity }>('/my-feature', { body: payload })
  }

  return {
    getEntity,
    list,
    create,
  }
}
```

**Composable Wrapper:**
```typescript
// areas/my-area/app/api/_composables_/index.ts
export { default as useApiMyFeature } from '../myFeatureApi'
export { default as useApiAnother } from '../anotherApi'
```

**Usage:**
```typescript
import { useApiMyFeature } from '#my-area/api'

const { getEntity, list } = useApiMyFeature()
```

### app/components/ Directory

**Purpose**: Vue components specific to the area

**Organization:**
- Group by feature/domain when >5 components
- Use kebab-case for directory names
- Use PascalCase for component file names

**Examples:**
```
components/
  # Flat structure (<5 components)
  base-card.vue
  loading-indicator.vue

  # Grouped by feature (>5 components)
  dashboard/
    dashboard-body/
      index.vue
    dashboard-header/
      index.vue
  site-details/
    base-layout/
      header-section.vue
    components/
      main-section/
        [many components]
```

**Naming:**
- Directories: `kebab-case` (e.g., `dashboard-body/`, `site-details/`)
- Files: `PascalCase.vue` or `kebab-case.vue` (project uses both, prefer PascalCase)

### app/composables/ Directory

**Purpose**: Vue composables (equivalent to React hooks)

**Naming:**
- `use` prefix (camelCase)
- Descriptive of what they do

**Examples:**
```
composables/
  useDashboard.ts
  useSiteDetailsActivity.ts
  useVariableLibrary.ts
  useAdvancedTargetRule.ts
```

**Pattern:**
```typescript
// areas/my-area/app/composables/useMyFeature.ts
export function useMyFeature() {
  const data = ref(null)
  const loading = ref(false)

  async function fetchData() {
    loading.value = true
    // ... fetch logic
    loading.value = false
  }

  return {
    data,
    loading,
    fetchData,
  }
}
```

### app/stores/ Directory

**Purpose**: Pinia stores for state management

**Naming:**
- camelCase with `.ts` extension
- Descriptive of the store's purpose

**Examples:**
```
stores/
  explorer.ts
  site-studio.ts
  whatsapp-templates.ts
```

**Pattern:**
```typescript
// areas/my-area/app/stores/myStore.ts
export const useMyStore = defineStore('my-store', () => {
  const state = ref({})

  function updateState(newState: any) {
    state.value = newState
  }

  return {
    state,
    updateState,
  }
})
```

### app/utils/ Directory

**Purpose**: Utility functions specific to the area

**Naming:**
- camelCase with `.ts` extension
- Descriptive of what they do

**Examples:**
```
utils/
  breadcrumb.ts
  code-editor.ts
  file-tree.ts
  format-text.ts
```

### constants/ Directory

**Purpose**: Constants and configuration values

**Location**: Root of area (not in `app/`)

**Examples:**
```
constants/
  sites.ts
  site-details.ts
  error-codes.ts
```

**Pattern:**
```typescript
// areas/my-area/constants/myFeature.ts
export const MY_FEATURE_ENDPOINT = '/my-feature'
export const MY_FEATURE_API_VERSION = 'v1'

export const MyFeatureStatus = {
  ACTIVE: 'active',
  INACTIVE: 'inactive',
} as const
```

### schemas/ Directory

**Purpose**: Zod validation schemas

**Location**: Root of area (not in `app/`)

**Examples:**
```
schemas/
  sites.ts
  site-settings.ts
  site-components.ts
```

**Pattern:**
```typescript
// areas/my-area/schemas/myFeature.ts
import { z } from 'zod'

export const CreateMyFeatureSchema = z.object({
  name: z.string().min(1),
  description: z.string().optional(),
})
```

### types/ Directory

**Purpose**: TypeScript type definitions

**Location**: Root of area (not in `app/`)

**Naming:**
- Use `.d.ts` extension for declaration files
- Or `.ts` for type files with exports

**Examples:**
```
types/
  sites.d.ts
  site-details.d.ts
  site-elements.d.ts
  common-response.d.ts
```

**Pattern:**
```typescript
// areas/my-area/types/myFeature.d.ts
export interface MyEntity {
  id: string
  name: string
  created_at: string
}

export interface CreateMyEntityPayload {
  name: string
  description?: string
}
```

---

## Import Aliases (Nuxt Configuration)

### Area-Specific Aliases

Each area defines aliases in `nuxt.config.ts`:

```typescript
// areas/my-area/nuxt.config.ts
alias: {
  '#my-area/types': join(currentDir, './types'),
  '#my-area/constants': join(currentDir, './constants'),
  '#my-area/schemas': join(currentDir, './schemas'),
  '#my-area/composables': join(appDir, './composables'),
  '#my-area/api': join(appDir, './api'),
  '#my-area': appDir,
}
```

### Usage Examples

```typescript
// ✅ PREFERRED - Use area aliases
import type { Site } from '#site-studio/types/sites'
import { SITE_API_BASE } from '#site-studio/constants/sites'
import { useApiSites } from '#site-studio/api'
import { useDashboard } from '#site-studio/composables/useDashboard'

// ✅ Cross-area imports (shared areas)
import { getErrorMessage } from '#common/utils/error'
import type { MediaTypes } from '#common/constants/media-config'

// ❌ AVOID - Relative paths from deep nesting
import { Site } from '../../../types/sites'
```

### Common Area Aliases

**Common Area** (`#common/`):
- `#common/utils/error` - Error utilities
- `#common/composables/useListData` - List data composable
- `#common/constants/` - Common constants

**Shared Areas** (e.g., `#form-shared/`, `#job-shared/`):
- Extendable by other areas
- Provide reusable functionality
- Follow same alias pattern: `#area-name/`

---

## File Naming Conventions

### Components

**Pattern**: PascalCase or kebab-case with `.vue` extension

```
MyComponent.vue          # Preferred
my-component.vue         # Also acceptable
DashboardHeader.vue
site-data-table.vue
```

**Avoid:**
- camelCase: `myComponent.vue` ❌
- All caps: `MYCOMPONENT.vue` ❌

### Composables

**Pattern**: camelCase with `use` prefix, `.ts` extension

```
useMyFeature.ts
useDashboard.ts
useSiteDetailsActivity.ts
useAdvancedTargetRule.ts
```

### API Services

**Pattern**: kebab-case or camelCase, `.ts` extension

```
myFeatureApi.ts
site-details.ts
site-components.ts
```

### Stores

**Pattern**: camelCase, `.ts` extension

```
explorer.ts
site-studio.ts
whatsapp-templates.ts
```

### Utils/Helpers

**Pattern**: camelCase, `.ts` extension

```
breadcrumb.ts
code-editor.ts
file-tree.ts
format-text.ts
```

### Types

**Pattern**: kebab-case, `.d.ts` or `.ts` extension

```
sites.d.ts
site-details.d.ts
common-response.d.ts
```

### Constants

**Pattern**: kebab-case or camelCase, `.ts` extension

```
sites.ts
site-details.ts
error-codes.ts
```

### Schemas

**Pattern**: kebab-case, `.ts` extension

```
sites.ts
site-settings.ts
site-components.ts
```

---

## Area Extension Pattern

### Extending Other Areas

Areas can extend other areas using Nuxt's `extends` feature:

```typescript
// areas/my-area/nuxt.config.ts
export default defineNuxtConfig({
  extends: [
    '@paradoxai/auth',        // Auth area
    '@paradoxai/common',      // Common utilities
    '@paradoxai/cem-shared',  // Shared CEM features
    '@paradoxai/job-shared',  // Shared job features
  ],
  // ... rest of config
})
```

**Benefits:**
- Access to extended area's components, composables, utils
- Shared functionality without duplication
- Clear dependency relationships

### Shared Areas Pattern

Areas ending with `-shared` are designed to be extended:

```
areas/
  form-shared/          # Shared form functionality
  job-shared/           # Shared job functionality
  candidate-shared/    # Shared candidate functionality
  messenger-shared/     # Shared messaging functionality
```

**When to create a shared area:**
- Functionality used by 3+ areas
- Domain-specific but reusable
- Needs to be versioned independently

---

## Import Organization

### Import Order (Recommended)

```typescript
// 1. Vue and Vue-related
import { ref, computed, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'

// 2. Nuxt composables
import { useAsyncData, useState } from '#imports'

// 3. Third-party libraries (alphabetical)
import { defu } from 'defu'
import type { SomeType } from 'some-library'

// 4. Area aliases (alphabetical by area)
import { useApiSites } from '#site-studio/api'
import { useDashboard } from '#site-studio/composables/useDashboard'
import { SITE_API_BASE } from '#site-studio/constants/sites'
import type { Site } from '#site-studio/types/sites'

// 5. Common/shared area imports
import { getErrorMessage } from '#common/utils/error'
import { useListData } from '#common/composables/useListData'

// 6. Relative imports (same area/feature)
import { MySubComponent } from './MySubComponent'
import { useMyFeature } from '../composables/useMyFeature'
```

**Use single quotes** for all imports (project standard)

---

## When to Create a New Area

### Create New Area When:

- **Independent domain**: Has its own business logic
- **Multiple related features**: 5+ components, multiple pages
- **Own API endpoints**: Distinct API surface
- **Reusable across apps**: Can be used in different contexts
- **Team ownership**: Owned by a specific team

**Examples:**
- `areas/site-studio/` - Site builder functionality
- `areas/admin/` - Admin panel features
- `areas/campaign/` - Campaign management

### Add to Existing Area When:

- **Related functionality**: Extends existing features
- **Same domain**: Logically belongs to the area
- **Shared API**: Uses same API endpoints
- **Small feature**: <5 components, single page

**Example:** Adding site settings to `areas/site-studio/`

### Create Shared Area When:

- **Used by 3+ areas**: Multiple areas need the functionality
- **Domain-specific but reusable**: Job, candidate, form features
- **Versioning needed**: Needs independent versioning
- **Clear boundaries**: Well-defined API surface

**Examples:**
- `areas/form-shared/` - Form builder shared across areas
- `areas/job-shared/` - Job-related shared functionality
- `areas/candidate-shared/` - Candidate-related shared functionality

---

## Testing Structure

### Test Organization

```
app/tests/
  __mocks__/           # Mock implementations
    useForm.mock.ts
    useSiteDetailsApi.mock.ts
  
  components/          # Component tests
    dashboard/
      dashboard-body.test.ts
  
  composables/         # Composable tests
    useDashboard.test.ts
    useVariableLibrary.test.ts
  
  fixtures/            # Test fixtures/data
    site-domain.fixture.ts
    site-elements.fixture.ts
  
  helpers/             # Test helpers
    mount.ts
    global-mocks.ts
  
  stores/              # Store tests
    explorer.test.ts
  
  utils/               # Utility tests
    breadcrumb.test.ts
    file-tree.test.ts
```

**Naming:**
- Test files: `*.test.ts` or `*.spec.ts`
- Fixtures: `*.fixture.ts`
- Mocks: `*.mock.ts`

---

## Directory Structure Visualization

```
areas/
├── admin/                    # Admin area
│   ├── app/
│   │   ├── api/
│   │   ├── components/
│   │   ├── composables/
│   │   ├── layouts/
│   │   ├── pages/
│   │   ├── stores/
│   │   └── utils/
│   ├── constants/
│   ├── schemas/
│   ├── types/
│   └── nuxt.config.ts
│
├── site-studio/              # Site Studio area
│   ├── app/
│   │   ├── api/
│   │   │   ├── _composables_/
│   │   │   └── sites.ts
│   │   ├── components/
│   │   ├── composables/
│   │   ├── pages/
│   │   └── stores/
│   ├── constants/
│   ├── schemas/
│   ├── types/
│   └── nuxt.config.ts
│
├── common/                   # Common/shared utilities
│   └── app/
│       ├── api/
│       ├── components/
│       ├── composables/
│       └── utils/
│
└── form-shared/              # Shared form functionality
    ├── app/
    ├── constants/
    ├── schemas/
    └── types/
```

---

## Best Practices

### 1. Area Organization

- Keep areas focused on a single domain
- Use shared areas for cross-cutting concerns
- Minimize dependencies between areas
- Each area should be independently testable

### 2. File Naming

- Components: PascalCase or kebab-case
- Composables: camelCase with `use` prefix
- Utils/Stores: camelCase
- Types: kebab-case with `.d.ts`
- Constants/Schemas: kebab-case

### 3. Import Aliases

- Always use area aliases (`#area-name/`)
- Prefer aliases over relative paths
- Group imports by area
- Use `#common/` for shared utilities

### 4. API Services

- One file per API domain
- Default export function pattern
- Export composables from `_composables_/index.ts`
- Type all parameters and returns

### 5. Component Organization

- Group by feature when >5 components
- Use descriptive directory names (kebab-case)
- Keep components focused and small
- Extract reusable logic to composables

### 6. State Management

- Use Pinia stores for shared state
- Use `useState` for SSR state
- Use `ref` for component-local state
- Keep stores focused and small

---

## Summary

**Key Principles:**

1. **Areas are self-contained**: Each area has its own app/, constants/, schemas/, types/
2. **Use area aliases**: `#area-name/` for clean imports
3. **Extend shared areas**: Use `extends` in nuxt.config.ts
4. **Consistent naming**: Follow naming conventions per file type
5. **Organize by feature**: Group components/composables by domain
6. **API composables**: Export from `_composables_/index.ts`

**Area Structure:**
- `app/` - Application code (api, components, composables, pages, stores, utils)
- `constants/` - Constants and configuration
- `schemas/` - Validation schemas
- `types/` - TypeScript types
- `nuxt.config.ts` - Area configuration with aliases

**See Also:**
- [data-fetching.md](data-fetching.md) - API service patterns
- Nuxt 3 Docs: [Layers](https://nuxt.com/docs/getting-started/layers)
- Nuxt 3 Docs: [Directory Structure](https://nuxt.com/docs/guide/directory-structure)
