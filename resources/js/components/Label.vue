<template>
    <label
        class="block text-sm leading-5 font-medium text-gray-700 dark:text-gray-300"
    >
        <slot></slot>
        <span v-if="required" class="ml-0.5 text-red-500" aria-hidden="true"
            >*</span
        >
    </label>
</template>
<script>
export default {
    props: {
        required: {
            type: Boolean,
            default: false,
        },
    },
}
</script>
