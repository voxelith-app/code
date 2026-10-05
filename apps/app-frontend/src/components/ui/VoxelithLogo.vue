<template>
	<img
		:src="isDarkTheme(theme.active) ? logo : icon"
		alt="Voxelith"
		draggable="false"
		class="object-contain"
	/>
</template>

<script setup lang="ts">
import { isDarkTheme, useTheme } from '@/composables/use-theme.ts'

import icon from '../../../../../branding/voxelith-icon.png'
import logo from '../../../../../branding/voxelith-logo.png'

const theme = useTheme()
</script>
