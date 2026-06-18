<template>
    <a
        :href="group.url"
        class="card flex w-full justify-between gap-x-2 rounded-md border border-gray-200 bg-white p-3 text-base text-gray-900 shadow duration-200 outline-none hover:border-indigo-500 focus:border-indigo-500 sm:w-72 dark:border-gray-700 dark:bg-gray-700 dark:text-gray-100"
        :class="isDragOver ? 'border-indigo-500 ring-2 ring-indigo-500' : ''"
        @dragover.prevent
        @dragenter="onDragEnter"
        @dragleave="onDragLeave"
        @drop.prevent="onDrop"
    >
        {{ group.name }}
        <span class="flex flex-wrap items-start gap-x-1.5">
            <span
                v-if="group.users_count > 1"
                class="flex items-center gap-x-0.5 rounded-md bg-gray-100 p-0.5 px-1.5 text-sm text-gray-600 dark:bg-gray-600 dark:text-gray-300"
            >
                <heroicons-user-icon class="h-3.5 w-3.5" />{{
                    group.users_count
                }}
            </span>
            <span
                v-if="group.children_count > 0"
                class="flex items-center gap-x-0.5 rounded-md bg-gray-100 p-0.5 px-1.5 text-sm text-gray-600 dark:bg-gray-600 dark:text-gray-300"
            >
                <heroicons-folder-open-icon class="h-3.5 w-3.5" />{{
                    group.children_count
                }}
            </span>
            <span
                class="flex items-center gap-x-0.5 rounded-md bg-gray-100 p-0.5 px-1.5 text-sm text-gray-600 dark:bg-gray-600 dark:text-gray-300"
            >
                <heroicons-key-icon class="h-3.5 w-3.5" />{{
                    group.credentials_count
                }}
            </span>
        </span>
    </a>
</template>

<script setup lang="ts">
import { useCredentialDrop } from '../composables/useCredentialDrop.js'

interface SubGroup {
    id: number
    name: string
    url: string
    users_count: number
    children_count: number
    credentials_count: number
}

const props = defineProps<{ group: SubGroup }>()

const { isDragOver, onDragEnter, onDragLeave, onDrop } = useCredentialDrop(
    props.group.id,
)
</script>
