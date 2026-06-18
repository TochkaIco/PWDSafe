<template>
    <div class="flex h-full flex-col px-2 py-4">
        <div
            v-if="loading"
            class="px-3 py-2 text-sm text-gray-400 dark:text-gray-500"
        >
            Loading…
        </div>

        <div v-else-if="error" class="px-3 py-2 text-sm text-red-500">
            Could not load groups.
        </div>

        <template v-else>
            <!-- Private section -->
            <div class="mb-4">
                <div class="mb-1 flex items-center justify-between px-3">
                    <a
                        v-if="privateGroup"
                        :href="privateGroup.url"
                        class="-mx-1 flex items-center gap-x-1.5 rounded px-1 text-xs font-semibold tracking-wider uppercase transition duration-150"
                        :class="[
                            activeGroupId === privateGroup.id
                                ? 'text-indigo-600 dark:text-indigo-400'
                                : 'text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-300',
                            privateRootDragOver
                                ? 'text-indigo-600 ring-2 ring-indigo-500 dark:text-indigo-400'
                                : '',
                        ]"
                        @dragover.prevent
                        @dragenter="privateRootOnDragEnter"
                        @dragleave="privateRootOnDragLeave"
                        @drop.prevent="privateRootOnDrop"
                    >
                        <heroicons-lock-closed-icon
                            class="h-3.5 w-3.5 flex-shrink-0"
                        />
                        Private
                    </a>
                    <span
                        v-else
                        class="text-xs font-semibold tracking-wider text-gray-400 uppercase dark:text-gray-500"
                    >
                        Private
                    </span>
                    <a
                        v-if="privateGroup"
                        :href="
                            '/groups/' + privateGroup.id + '/subgroups/create'
                        "
                        class="text-gray-400 transition duration-150 hover:text-indigo-500 dark:text-gray-500 dark:hover:text-indigo-400"
                        title="New sub-group"
                    >
                        <heroicons-folder-plus-icon class="h-4 w-4" />
                    </a>
                </div>
                <!-- Children render directly under the label; no tree item for the private root -->
                <vault-sidebar-item
                    v-for="child in privateGroup?.children ?? []"
                    :key="child.id"
                    :node="child"
                    :active-group-id="activeGroupId"
                />
            </div>

            <!-- Shared section -->
            <div>
                <div class="mb-1 flex items-center justify-between px-3">
                    <span
                        class="text-xs font-semibold tracking-wider text-gray-400 uppercase dark:text-gray-500"
                    >
                        Shared
                    </span>
                    <a
                        href="/groups/create"
                        class="text-gray-400 transition duration-150 hover:text-indigo-500 dark:text-gray-500 dark:hover:text-indigo-400"
                        title="New group"
                    >
                        <heroicons-folder-plus-icon class="h-4 w-4" />
                    </a>
                </div>
                <p
                    v-if="sharedGroups.length === 0"
                    class="px-3 text-xs text-gray-400 italic dark:text-gray-500"
                >
                    No shared groups yet.
                </p>
                <vault-sidebar-item
                    v-for="group in sharedGroups"
                    :key="group.id"
                    :node="group"
                    :active-group-id="activeGroupId"
                />
            </div>

            <!-- Bottom links -->
            <div
                class="mt-auto border-t border-gray-200 pt-4 dark:border-gray-600"
            >
                <a
                    href="/securitycheck"
                    class="flex items-center gap-x-2 rounded-md px-3 py-1.5 text-sm text-gray-600 transition duration-150 hover:bg-gray-100 dark:text-gray-400 dark:hover:bg-gray-600"
                >
                    <ShieldCheckIcon class="h-4 w-4 flex-shrink-0" />
                    Security check
                </a>
            </div>
        </template>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { ShieldCheckIcon } from '@heroicons/vue/24/outline'
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
    activeGroupId: number | null
}>()

const loading = ref(true)
const error = ref(false)
const privateGroup = ref<SidebarNode | null>(null)
const sharedGroups = ref<SidebarNode[]>([])

const loadSidebar = async () => {
    try {
        const response = await fetch('/api/sidebar', {
            headers: { 'X-Requested-With': 'XMLHttpRequest' },
        })
        if (!response.ok) {
            throw new Error('Failed to load sidebar')
        }
        const data = await response.json()
        privateGroup.value = data.private
        sharedGroups.value = data.shared
    } catch {
        error.value = true
    } finally {
        loading.value = false
    }
}

onMounted(() => {
    loadSidebar()
    window.addEventListener('credential-moved', loadSidebar)
})

onUnmounted(() => {
    window.removeEventListener('credential-moved', loadSidebar)
})

const privateGroupId = computed(() => privateGroup.value?.id ?? 0)
const {
    isDragOver: privateRootDragOver,
    onDragEnter: privateRootOnDragEnter,
    onDragLeave: privateRootOnDragLeave,
    onDrop: privateRootOnDrop,
} = useCredentialDrop(privateGroupId)
</script>
