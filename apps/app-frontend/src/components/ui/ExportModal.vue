<script setup>
import { FolderOpenIcon, XIcon } from '@modrinth/assets'
import {
	Button,
	commonMessages,
	defineMessages,
	FileTreeSelect,
	injectNotificationManager,
	injectPopupNotificationManager,
	Input,
	NewModal,
	Textarea,
	useVIntl,
} from '@modrinth/ui'
import { save } from '@tauri-apps/plugin-dialog'
import { computed, ref, shallowRef } from 'vue'

import { PackageIcon } from '@/assets/icons'
import { export_instance_mrpack, get_pack_export_candidates } from '@/helpers/instance'
import { highlightInFolder } from '@/helpers/utils'

const { handleError } = injectNotificationManager()
const popupNotificationManager = injectPopupNotificationManager()
const { formatMessage } = useVIntl()

const messages = defineMessages({
	header: { id: 'app.export-modal.header', defaultMessage: 'Export modpack' },
	mobileHeader: { id: 'app.export-modal.mobile-header', defaultMessage: 'Export for mobile' },
	mobileHint: {
		id: 'app.export-modal.mobile-hint',
		defaultMessage:
			'Shaders and mods that usually do not work on phones (Iris, Oculus, Distant Horizons) are left out. On the phone, import the file in Voxelith Launcher and tap Optimize.',
	},
	mobileNameSuffix: { id: 'app.export-modal.mobile-name-suffix', defaultMessage: '(mobile)' },
	modpackNameLabel: { id: 'app.export-modal.modpack-name-label', defaultMessage: 'Modpack name' },
	modpackNamePlaceholder: {
		id: 'app.export-modal.modpack-name-placeholder',
		defaultMessage: 'Modpack name',
	},
	versionNumberLabel: {
		id: 'app.export-modal.version-number-label',
		defaultMessage: 'Version number',
	},
	versionNumberPlaceholder: {
		id: 'app.export-modal.version-number-placeholder',
		defaultMessage: '1.0.0',
	},
	descriptionPlaceholder: {
		id: 'app.export-modal.description-placeholder',
		defaultMessage: 'Enter modpack description...',
	},
	exportButton: { id: 'app.export-modal.export-button', defaultMessage: 'Export' },
	exportComplete: {
		id: 'app.export-modal.export-complete',
		defaultMessage: 'Export complete',
	},
	exportCompleteDescription: {
		id: 'app.export-modal.export-complete-description',
		defaultMessage: '{name} was exported successfully.',
	},
})

const props = defineProps({
	instance: {
		type: Object,
		required: true,
	},
})

/** Mods that are heavy or crash on Android launchers, matched by file name. */
const MOBILE_EXCLUDED_MODS = /^(iris|oculus|distant[-_ ]?horizons|physics[-_ ]?mod)/i
const MOBILE_EXCLUDED_FOLDERS = ['shaderpacks']

defineExpose({
	show: (options = {}) => {
		mobile.value = options.mobile === true
		resetExportState()
		exportModal.value.show()
		void initFiles().catch(handleError)
	},
})

const exportModal = ref(null)
const mobile = ref(false)
const header = computed(() => formatMessage(mobile.value ? messages.mobileHeader : messages.header))
const nameInput = ref(props.instance.name)
const exportDescription = ref('')
const versionInput = ref('1.0.0')
const files = shallowRef([])
const includedFilePaths = ref([])
const excludedFilePaths = ref([])
const fileTreeKey = ref(0)
const filesLoadId = ref(0)
const directoryEntries = new Map()
const currentDirectory = ref('')

async function initFiles() {
	const loadId = ++filesLoadId.value
	const exportCandidates = await get_pack_export_candidates(props.instance.id)
	if (loadId !== filesLoadId.value) return

	files.value = exportCandidates
	directoryEntries.set('', exportCandidates)
	currentDirectory.value = ''
	includedFilePaths.value = files.value
		.filter((file) => !file.disabled && file.defaultSelected)
		.map((file) => file.path)
	if (mobile.value) await applyMobileDefaults(loadId)
}

async function applyMobileDefaults(loadId) {
	includedFilePaths.value = includedFilePaths.value.filter(
		(path) => !MOBILE_EXCLUDED_FOLDERS.includes(path),
	)
	const excluded = [...MOBILE_EXCLUDED_FOLDERS]
	if (files.value.some((file) => file.path === 'mods')) {
		const mods = await get_pack_export_candidates(props.instance.id, 'mods')
		if (loadId !== filesLoadId.value) return
		directoryEntries.set('mods', mods)
		for (const mod of mods) {
			if (MOBILE_EXCLUDED_MODS.test(mod.path.split('/').pop() ?? '')) excluded.push(mod.path)
		}
	}
	excludedFilePaths.value = excluded
	fileTreeKey.value += 1
}

const exportPack = async () => {
	const outputPath = await save({
		defaultPath: `${nameInput.value} ${versionInput.value}.mrpack`,
		filters: [
			{
				name: 'Modrinth Modpack',
				extensions: ['mrpack'],
			},
		],
	})

	if (outputPath) {
		exportModal.value.hide()

		try {
			await export_instance_mrpack(
				props.instance.id,
				outputPath,
				includedFilePaths.value,
				excludedFilePaths.value,
				versionInput.value,
				exportDescription.value,
				nameInput.value,
			)

			const fileName = outputPath.split(/[\\/]/).pop() ?? outputPath
			popupNotificationManager.addPopupNotification({
				title: formatMessage(messages.exportComplete),
				text: formatMessage(messages.exportCompleteDescription, { name: fileName }),
				type: 'success',
				buttons: [
					{
						label: formatMessage(commonMessages.openInFolderButton),
						icon: FolderOpenIcon,
						action: () => highlightInFolder(outputPath).catch(handleError),
					},
				],
			})
		} catch (error) {
			handleError(error)
		}
	}
}

function resetExportState() {
	nameInput.value = mobile.value
		? `${props.instance.name} ${formatMessage(messages.mobileNameSuffix)}`
		: props.instance.name
	exportDescription.value = ''
	versionInput.value = '1.0.0'
	files.value = []
	includedFilePaths.value = []
	excludedFilePaths.value = []
	fileTreeKey.value += 1
	directoryEntries.clear()
	currentDirectory.value = ''
}

async function loadExportDirectory(path) {
	const normalizedPath = normalizeExportPath(path)
	currentDirectory.value = normalizedPath

	const cachedEntries = directoryEntries.get(normalizedPath)
	if (cachedEntries) {
		files.value = cachedEntries
		return
	}

	const loadId = filesLoadId.value
	files.value = []

	try {
		const childItems = await get_pack_export_candidates(
			props.instance.id,
			normalizedPath || undefined,
		)
		if (loadId !== filesLoadId.value) return

		directoryEntries.set(normalizedPath, childItems)
		if (currentDirectory.value === normalizedPath) {
			files.value = childItems
		}
	} catch {
		if (currentDirectory.value === normalizedPath) files.value = []
	}
}

function normalizeExportPath(path) {
	return path.replaceAll('\\', '/').split('/').filter(Boolean).join('/')
}
</script>

<template>
	<NewModal
		ref="exportModal"
		:header="header"
		scrollable
		width="46rem"
		max-width="calc(100vw - 2rem)"
	>
		<div class="flex flex-col gap-4">
			<p v-if="mobile" class="m-0 text-secondary">{{ formatMessage(messages.mobileHint) }}</p>
			<div class="grid grid-cols-2 gap-4">
				<div class="labeled_input w-full">
					<p class="text-contrast font-semibold">{{ formatMessage(messages.modpackNameLabel) }}</p>
					<Input
						v-model="nameInput"
						type="text"
						:placeholder="formatMessage(messages.modpackNamePlaceholder)"
						clearable
						wrapper-class="w-full"
					/>
				</div>
				<div class="labeled_input w-full">
					<p class="text-contrast font-semibold">
						{{ formatMessage(messages.versionNumberLabel) }}
					</p>
					<Input
						v-model="versionInput"
						type="text"
						:placeholder="formatMessage(messages.versionNumberPlaceholder)"
						clearable
						wrapper-class="w-full"
					/>
				</div>
			</div>
			<div class="flex flex-col gap-2 min-w-0">
				<p class="m-0 text-contrast font-semibold">
					{{ formatMessage(commonMessages.descriptionLabel) }}
				</p>
				<Textarea
					v-model="exportDescription"
					:placeholder="formatMessage(messages.descriptionPlaceholder)"
					wrapper-class="w-full"
				/>
			</div>
			<FileTreeSelect
				:key="fileTreeKey"
				v-model="includedFilePaths"
				v-model:excluded-paths="excludedFilePaths"
				class="min-w-0"
				:items="files"
				lazy
				@navigate="loadExportDirectory"
			/>
		</div>
		<template #actions>
			<div class="flex items-center justify-end gap-2">
				<Button type="outlined" @click="exportModal.hide">
					<XIcon />
					{{ formatMessage(commonMessages.cancelButton) }}
				</Button>
				<Button type="colored" color="brand" @click="exportPack">
					<PackageIcon />
					{{ formatMessage(messages.exportButton) }}
				</Button>
			</div>
		</template>
	</NewModal>
</template>
