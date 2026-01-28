# @paradoxai/ui Component Reference

Complete catalog of @paradoxai/ui components with usage patterns for Nuxt 3 projects.

## Overview

This project uses the **@paradoxai/ui** component library, which provides components prefixed with `Ol` (e.g., `OlButton`, `OlDialog`, `OlInput`). Components are auto-imported in Nuxt 3, so no explicit imports are needed.

**Key Technologies:**
- Nuxt 3 with Vue 3 Composition API
- TypeScript
- UnoCSS for styling (use `:uno:` prefix)
- Element Plus integration
- Auto-imported components

## Component Usage Pattern

Components are automatically available in templates. Use the `Ol` prefix:

```vue
<script setup lang="ts">
// Components are auto-imported, no need to import manually
const dialogVisible = ref(false)
</script>

<template>
  <OlButton @click="dialogVisible = true">Open Dialog</OlButton>
  <OlDialog v-model="dialogVisible" title="Example">
    Content here
  </OlDialog>
</template>
```

## Form & Input Components

### Button (ElButton)

While `OlButton` exists, the project primarily uses Element Plus `ElButton`:

```vue
<template>
  <ElButton type="primary" size="large">Submit</ElButton>
  <ElButton type="default" @click="handleCancel">Cancel</ElButton>
  <ElButton type="text" :icon="Icon">Icon Button</ElButton>
</template>
```

Types: `primary | default | success | warning | danger | info | text`
Sizes: `large | default | small`

### Input

```vue
<script setup lang="ts">
const inputValue = ref('')
</script>

<template>
  <OlInput
    v-model="inputValue"
    placeholder="Enter text"
    size="large"
  />
  
  <OlInput
    v-model="inputValue"
    placeholder="With prefix icon"
    :prefix-icon="'i-ol-search'"
  />
  
  <OlInput
    v-model="inputValue"
    placeholder="With suffix"
    :suffix-icon="'i-ol-close'"
    :error="hasError"
  />
</template>
```

Props: `size`, `placeholder`, `disabled`, `readonly`, `error`, `prefixIcon`, `suffixIcon`, `rounded`, `transparent`, `fitContent`

### Form (with OlForm + OlFormItem)

```vue
<script setup lang="ts">
import { z } from 'zod'

const formData = ref({
  username: '',
  email: '',
})

const formRules = {
  username: [
    { required: true, message: 'Username is required', trigger: 'blur' },
  ],
  email: [
    { required: true, message: 'Email is required', trigger: 'blur' },
    { type: 'email', message: 'Invalid email', trigger: 'blur' },
  ],
}
</script>

<template>
  <OlForm
    ref="formRef"
    :model="formData"
    :rules="formRules"
  >
    <OlFormItem
      label="Username"
      prop="username"
      class=":uno: w-80 flex-col gap-1"
    >
      <OlInput v-model="formData.username" />
    </OlFormItem>
    
    <OlFormItem
      label="Email"
      prop="email"
      class=":uno: w-80 flex-col gap-1"
    >
      <OlInput v-model="formData.email" type="email" />
    </OlFormItem>
  </OlForm>
</template>
```

### Select

```vue
<script setup lang="ts">
const selectedValue = ref('')
const options = [
  { label: 'Option 1', value: 'opt1' },
  { label: 'Option 2', value: 'opt2' },
]
</script>

<template>
  <OlSelect
    v-model="selectedValue"
    placeholder="Select an option"
    :options="options"
    filterable
  />
  
  <OlSelect
    v-model="selectedValue"
    placeholder="Multiple selection"
    multiple
    :options="options"
  />
</template>
```

Props: `v-model`, `options`, `placeholder`, `multiple`, `filterable`, `disabled`, `size`, `useBottomsheet` (for mobile)

### Checkbox Group

```vue
<script setup lang="ts">
const checkedItems = ref([])
</script>

<template>
  <OlCheckboxGroup v-model="checkedItems">
    <ElCheckbox label="Option 1" value="opt1" />
    <ElCheckbox label="Option 2" value="opt2" />
    <ElCheckbox label="Option 3" value="opt3" />
  </OlCheckboxGroup>
</template>
```

### Radio Group

```vue
<script setup lang="ts">
const selectedRadio = ref('')
</script>

<template>
  <OlRadioGroup v-model="selectedRadio">
    <ElRadio label="Option One" value="option-one" />
    <ElRadio label="Option Two" value="option-two" />
  </OlRadioGroup>
</template>
```

### Textarea

```vue
<script setup lang="ts">
const textareaValue = ref('')
</script>

<template>
  <ElInput
    v-model="textareaValue"
    type="textarea"
    :rows="4"
    placeholder="Type your message here"
  />
</template>
```

### Switch

```vue
<script setup lang="ts">
const switchValue = ref(false)
</script>

<template>
  <OlSwitch v-model="switchValue" />
</template>
```

### Date Picker

```vue
<script setup lang="ts">
import { format } from 'date-fns'

const date = ref<Date>()
</script>

<template>
  <OlDatePicker
    v-model="date"
    type="date"
    placeholder="Pick a date"
  />
  
  <OlDatePicker
    v-model="date"
    type="daterange"
    range-separator="To"
    start-placeholder="Start date"
    end-placeholder="End date"
  />
</template>
```

## Layout & Navigation

### Card

```vue
<template>
  <OlCard
    shadow="never"
    :body-class="':uno: p-6'"
    class=":uno: border-grey-400"
  >
    <template #header>
      <div class=":uno: text-xl font-semibold">Card Title</div>
    </template>
    
    <div>Card Content</div>
    
    <template #footer>
      <ElButton>Action</ElButton>
    </template>
  </OlCard>
</template>
```

Props: `shadow` (`never | always | hover`), `bodyClass`, `headerClass`, `footerClass`, `disabled`

### Tabs

```vue
<script setup lang="ts">
const activeTab = ref('account')
</script>

<template>
  <OlTabs v-model="activeTab">
    <ElTabPane label="Account" name="account">
      Account settings
    </ElTabPane>
    <ElTabPane label="Password" name="password">
      Password settings
    </ElTabPane>
  </OlTabs>
</template>
```

Props: `v-model`, `type`, `size`, `underline`, `variant`

### Accordion (Collapse)

```vue
<script setup lang="ts">
const activeNames = ref(['item-1'])
</script>

<template>
  <OlCollapse v-model="activeNames">
    <OlCollapseItem title="Is it accessible?" name="item-1">
      Yes. It adheres to WAI-ARIA design pattern.
    </OlCollapseItem>
    <OlCollapseItem title="Is it styled?" name="item-2">
      Yes. Comes with default styles customizable with UnoCSS.
    </OlCollapseItem>
  </OlCollapse>
</template>
```

### Menu

```vue
<template>
  <OlMenu>
    <OlMenuItem index="1">Getting Started</OlMenuItem>
    <OlSubMenu index="2">
      <template #title>Components</template>
      <OlMenuItem index="2-1">Introduction</OlMenuItem>
      <OlMenuItem index="2-2">Installation</OlMenuItem>
    </OlSubMenu>
  </OlMenu>
</template>
```

## Overlays & Dialogs

### Dialog

```vue
<script setup lang="ts">
const dialogVisible = ref(false)
</script>

<template>
  <ElButton @click="dialogVisible = true">Open Dialog</ElButton>
  
  <OlDialog
    v-model="dialogVisible"
    title="Are you sure?"
    width="30rem"
    type="primary"
    align-center
    append-to-body
    @close="dialogVisible = false"
    @confirm="handleConfirm"
  >
    <template #body>
      <p>This action cannot be undone.</p>
    </template>
  </OlDialog>
</template>
```

Props: `v-model`, `title`, `width`, `type` (`primary | danger`), `align-center`, `append-to-body`, `text-confirm`, `text-cancel`, `disable-confirm`

Events: `@close`, `@confirm`, `@cancel`

### Drawer

```vue
<script setup lang="ts">
const drawerVisible = ref(false)
</script>

<template>
  <ElButton @click="drawerVisible = true">Open Drawer</ElButton>
  
  <OlDrawer
    v-model="drawerVisible"
    title="Title"
    size="38rem"
    @close="drawerVisible = false"
  >
    <template #default>
      <p>Drawer content</p>
    </template>
    
    <template #footer>
      <ElButton @click="drawerVisible = false">Cancel</ElButton>
      <ElButton type="primary" @click="handleSubmit">Submit</ElButton>
    </template>
  </OlDrawer>
</template>
```

Props: `v-model`, `title`, `size`, `direction` (`rtl | ltr | ttb | btt`)

### Popover

```vue
<template>
  <OlPopover
    placement="bottom-end"
    :offset="8"
    trigger="hover"
    :width="278"
    :show-arrow="false"
  >
    <template #reference>
      <ElButton>Hover me</ElButton>
    </template>
    
    <div>Popover content here</div>
  </OlPopover>
</template>
```

Props: `placement`, `trigger` (`hover | click | focus`), `width`, `show-arrow`, `offset`

### Dropdown

```vue
<script setup lang="ts">
const menuOptions = [
  { label: 'Option 1', command: 'opt1' },
  { label: 'Option 2', command: 'opt2' },
]

function handleCommand(command: string) {
  console.log(command)
}
</script>

<template>
  <OlDropdown
    trigger="click"
    placement="bottom-end"
    :options="menuOptions"
    @command="handleCommand"
  >
    <ElButton>
      <template #icon>
        <i class=":uno: i-ol-gear size-7" />
      </template>
    </ElButton>
  </OlDropdown>
</template>
```

### Notification (OlNotify)

```vue
<script setup lang="ts">
import { h } from 'vue'

function showNotification() {
  OlNotify({
    type: 'success',
    icon: () => h('i', { class: ':uno: i-ol-check icon:mr-2' }),
    message: 'Operation successful',
  })
}

function showError() {
  OlNotify({
    type: 'error',
    icon: () => h('i', { class: ':uno: i-ol-alert-caution icon:mr-2' }),
    message: 'An error occurred',
  })
}
</script>
```

Types: `success | warning | info | error`

### Command (Select with Search)

```vue
<script setup lang="ts">
const searchValue = ref('')
const filteredOptions = computed(() => {
  // Filter logic
  return options.value.filter(opt => 
    opt.label.toLowerCase().includes(searchValue.value.toLowerCase())
  )
})
</script>

<template>
  <OlSelectList
    v-model="selectedValue"
    :options="filteredOptions"
    filterable
    :show-input-search="true"
  />
</template>
```

## Feedback & Status

### Alert (ElAlert)

```vue
<template>
  <ElAlert
    title="Heads up!"
    description="You can add components using the component library."
    type="info"
  />
  
  <ElAlert
    title="Error"
    description="Session expired. Please log in."
    type="error"
    :closable="true"
  />
</template>
```

Types: `success | warning | info | error`

### Progress

```vue
<template>
  <OlProgress :percentage="33" />
  
  <OlProgress
    :percentage="66"
    :status="'success'"
    :stroke-width="8"
  />
</template>
```

### Skeleton (ElSkeleton)

```vue
<template>
  <ElSkeleton :rows="3" animated />
  
  <div class=":uno: flex items-center space-x-4">
    <ElSkeleton
      :width="48"
      :height="48"
      shape="circle"
    />
    <div class=":uno: space-y-2">
      <ElSkeleton :width="250" :height="16" />
      <ElSkeleton :width="200" :height="16" />
    </div>
  </div>
</template>
```

## Display Components

### Table

```vue
<script setup lang="ts">
const tableData = ref([
  { invoice: 'INV001', status: 'Paid', amount: '$250.00' },
  { invoice: 'INV002', status: 'Pending', amount: '$150.00' },
])

const columns = [
  { prop: 'invoice', label: 'Invoice' },
  { prop: 'status', label: 'Status' },
  { prop: 'amount', label: 'Amount' },
]
</script>

<template>
  <OlTable
    :data="tableData"
    :columns="columns"
    stripe
  >
    <template #empty>
      <div>No data available</div>
    </template>
  </OlTable>
</template>
```

### Avatar

```vue
<template>
  <ElAvatar src="https://example.com/avatar.png" />
  
  <ElAvatar>
    <span>CN</span>
  </ElAvatar>
</template>
```

### Badge

```vue
<template>
  <OlBadge :value="5" type="error">
    <ElButton>Notifications</ElButton>
  </OlBadge>
  
  <OlBadge
    :is-dot="true"
    type="error"
  >
    <ElButton>Messages</ElButton>
  </OlBadge>
  
  <OlBadge
    :value="badge"
    :max="99"
    variant="standard"
    type="error"
    circle
  >
    <ElButton>Filters</ElButton>
  </OlBadge>
</template>
```

Props: `value`, `max`, `is-dot`, `type` (`success | warning | danger | info`), `variant`, `circle`

### Tag

```vue
<template>
  <OlTag>Default</OlTag>
  <OlTag type="success">Success</OlTag>
  <OlTag type="warning">Warning</OlTag>
  <OlTag type="danger">Danger</OlTag>
  <OlTag closable @close="handleClose">Closable</OlTag>
</template>
```

## Advanced Components

### Editor

```vue
<script setup lang="ts">
const content = ref('')
</script>

<template>
  <OlEditor
    v-model="content"
    :editor-style="{ minHeight: '400px' }"
    placeholder="Start typing..."
  />
</template>
```

### Upload

```vue
<script setup lang="ts">
const fileList = ref([])

function handleUpload(file: File) {
  // Upload logic
}
</script>

<template>
  <OlUpload
    v-model:file-list="fileList"
    action="/api/upload"
    :on-success="handleUpload"
  >
    <ElButton>Click to upload</ElButton>
  </OlUpload>
</template>
```

### Loading

```vue
<script setup lang="ts">
const loading = ref(false)

function showLoading() {
  OlLoading.service({
    text: 'Loading...',
    background: 'rgba(0, 0, 0, 0.7)',
  })
}
</script>
```

### Infinite Loading

```vue
<template>
  <OlInfiniteLoading
    :loading="isLoading"
    :finished="isFinished"
    @load="loadMore"
  >
    <template #finished>
      <span>No more data</span>
    </template>
  </OlInfiniteLoading>
</template>
```

## Styling with UnoCSS

All components support UnoCSS classes. Use the `:uno:` prefix for utility classes:

```vue
<template>
  <OlCard
    class=":uno: border-grey-400 rounded-lg"
    :body-class=":uno: p-6"
  >
    <div class=":uno: flex items-center gap-2">
      <ElButton class=":uno: px-4 py-2">Button</ElButton>
    </div>
  </OlCard>
</template>
```

## Component Composition

Components can be composed together:

```vue
<template>
  <OlCard class=":uno: max-w-md">
    <OlForm :model="formData">
      <OlFormItem label="Name" prop="name">
        <OlInput v-model="formData.name" />
      </OlFormItem>
      
      <OlFormItem>
        <ElButton type="primary" @click="handleSubmit">
          Submit
        </ElButton>
      </OlFormItem>
    </OlForm>
  </OlCard>
</template>
```

## Notes

- All `Ol` components are auto-imported in Nuxt 3
- Element Plus components (`El*`) are also available
- Use TypeScript for type safety
- Components follow Vue 3 Composition API patterns
- Styling uses UnoCSS with `:uno:` prefix
- Many components support both desktop and mobile (bottomsheet) variants
