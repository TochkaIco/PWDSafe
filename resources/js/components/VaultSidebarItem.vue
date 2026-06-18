<template>
    <div>
        <div
            class="flex items-center gap-x-1"
            @dragover.prevent
            @dragenter="onDragEnter"
            @dragleave="onDragLeave"
            @drop.prevent="onDrop"
        >
            <!-- Expand/collapse chevron -->
            <button
                v-if="node.children.length > 0"
                @click.prevent="expanded = !expanded"
                class="h-4 w-4 flex-shrink-0 text-gray-400 hover:text-gray-600 dark:text-gray-500 dark:hover:text-gray-300"
            >
                <heroicons-chevron-down-icon v-if="expanded" class="h-4 w-4" />
                <heroicons-chevron-right-icon v-else class="h-4 w-4" />
            </button>
            <span v-else class="w-4 flex-shrink-0"></span>

            <!-- Folder icon + name link -->
            <a
                :href="node.url"
                class="flex min-w-0 flex-1 items-center gap-x-1.5 rounded-md px-3 py-1.5 text-sm font-medium"
                :class="[
                    isActive
                        ? 'bg-indigo-50 text-indigo-700 dark:bg-indigo-900/30 dark:text-indigo-300'
                        : 'text-gray-700 hover:bg-gray-100 dark:text-gray-300 dark:hover:bg-gray-600',
                    isDragOver ? 'ring-2 ring-indigo-500 ring-inset' : '',
                ]"
            >
                <heroicons-folder-open-icon
                    v-if="isActive"
                    class="h-4 w-4 flex-shrink-0"
                />
                <heroicons-folder-icon v-else class="h-4 w-4 flex-shrink-0" />
                <span class="truncate">{{ node.name }}</span>
                <div class="ml-auto flex flex-shrink-0 items-center gap-x-1.5">
                    <span
                        v-if="node.usersCount > 1"
                        class="flex items-center gap-x-0.5 text-xs text-gray-400 tabular-nums dark:text-gray-500"
                    >
                        <heroicons-user-icon class="h-3 w-3" />
                        {{ node.usersCount }}
                    </span>
                    <span
                        v-if="node.credentialsCount > 0"
                        class="ml-1 text-xs text-gray-400 tabular-nums dark:text-gray-500"
                        >{{ node.credentialsCount }}</span
                    >
                </div>
            </a>
        </div>

        <!-- Children -->
        <div v-if="expanded && node.children.length > 0" class="ml-4">
            <vault-sidebar-item
                v-for="child in node.children"
                :key="child.id"
                :node="child"
                :active-group-id="activeGroupId"
            />
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useCredentialDrop } from '../composables/useCredentialDrop.js'

interface SidebarNode {
    id: number
    name: string
    url: string
    credentialsCount: number
    usersCount: number
    children: SidebarNode[]
}

const props = defineProps<{
    node: SidebarNode
    activeGroupId: number | null
}>()

const isActive = computed(() => props.node.id === props.activeGroupId)

const hasActiveDescendant = (
    node: SidebarNode,
    targetId: number | null,
): boolean => {
    if (!targetId) {
        return false
    }
    return node.children.some(
        (child) =>
            child.id === targetId || hasActiveDescendant(child, targetId),
    )
}

const expanded = ref(
    isActive.value || hasActiveDescendant(props.node, props.activeGroupId),
)

const { isDragOver, onDragEnter, onDragLeave, onDrop } = useCredentialDrop(
    props.node.id,
)
</script>
