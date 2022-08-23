<template>
	<div
		class="emote-upload-wrap"
		@drop.prevent="onDropFile"
		@dragover.prevent
		@dragenter="onDragEnter"
		@dragleave="onDragLeave"
	>
		<!-- Drop Overlay -->
		<transition name="drop">
			<div v-if="dragOver" class="dropper">
				<h1>Drop here</h1>
				<font-awesome-icon class="icon" :icon="['fab', 'plus']" color="white" />
			</div>
		</transition>
		<!-- Load Overlay -->
		<transition name="load">
			<div v-if="uploadError || uploadProgress" class="loader">
				<h1>{{ uploadError ? "Error" : "Uploading..." }}</h1>
				<div v-if="uploadError" class="error-display">
					<h3>{{ uploadError }}</h3>
					<button @click="clearError">Ok</button>
				</div>
				<div v-else class="progress" :style="{ '--progress': uploadProgress }"></div>
			</div>
		</transition>

		<div class="emote-upload">
			<!-- Parent Emote -->
			<div v-if="parentEmote" class="side-parent">
				<h3>Parent Emote</h3>
				<div class="parent">
					<img
						alt="parent"
						:src="
							Emote.GetImage(Emote.GetCurrentVersion(parentEmote)?.images ?? [], ImageFormat.WEBP, '2x')
								?.url
						"
					/>
					<span class="name">{{ parentEmote.name }}</span>
				</div>
			</div>
			<!-- Emote Details -->
			<div class="side-details">
				<h3>Emote details</h3>
				<form class="details" @submit.prevent="upload">
					<TextInput v-model="form.name" class="form-item" :label="t(txt.inputEmoteName)" />
					<TextInput
						v-if="parentEmote"
						v-model="form.version_description"
						:label="t('emote.upload.version_description')"
					/>

					<Checkbox :checked="form.zero_width" label="Zero-Width" class="form-item" />
					<Checkbox :checked="form.private" label="Private" class="form-item" />

					<h4>{{ t("emote.tags") }}</h4>
					<EmoteTagList :editable="true" :limit="6" @update="(tags) => (form.tags = tags)" />

					<h4>Before you submit</h4>
					<p class="before-submit">
						Some long text xd I love 7tv maoE! Maybe something regarding private or unlisted or idk what to
						type here anyways this should be enough to trigger a linebreak and child thing !! Is the parent
						emote shown idk
						<i18n-t v-if="parentEmote" keypath="emote.upload.as_child" tag="p">
							<span style="font-weight: 600">{{ parentEmote.name }}</span>
						</i18n-t>
					</p>
					<input type="submit" :value="t('common.submit')" />
				</form>
			</div>
			<!-- Preview -->
			<div v-if="imgUrl" class="side-preview">
				<h3>Emote Preview</h3>
				<!-- Emote Preview -->
				<div class="emote-preview">
					<h5>{{ form.name }}</h5>
					<div class="sizes">
						<div class="size">
							<img class="s32" alt="emote" :src="imgUrl" />
							<span>{{ (imageAspectRatio * 32).toFixed(0) }}x32</span>
						</div>
						<div class="size">
							<img class="s64" alt="emote" :src="imgUrl" />
							<span>{{ (imageAspectRatio * 64).toFixed(0) }}x64</span>
						</div>
						<div class="size">
							<img class="s96" alt="emote" :src="imgUrl" />
							<span>{{ (imageAspectRatio * 96).toFixed(0) }}x96</span>
						</div>
						<div class="size">
							<img
								ref="aspectRef"
								alt="emote"
								:src="imgUrl"
								class="s128"
								@load="updateImageAspectRatio"
							/>
							<span>{{ (imageAspectRatio * 128).toFixed(0) }}x128</span>
						</div>
					</div>
				</div>
				<h3>Chat Preview</h3>
				<!-- Chat Preview -->
				<div class="chat-preview">
					<div class="line">
						<span class="user" :style="{ color: `#${userColor.toString(16).padStart(6, '0')}` }"
							>{{ userName }}:</span
						>
						Look at my new emote: <img class="emote" :alt="form.name" :src="imgUrl" />
					</div>
					<div class="line">
						<span class="user" :style="{ color: `#${userColor.toString(16).padStart(6, '0')}` }"
							>{{ userName }}:</span
						>
						<img class="emote" :alt="form.name" :src="imgUrl" />
					</div>
				</div>
			</div>
			<!-- Emote Input -->
			<div v-else class="side-add-emote">
				<h3>{{ t("emote.upload.image_upload") }}</h3>
				<div class="inner">
					<div class="image">
						<h5>Click or drag an image here.</h5>
						<input id="file-upload" hidden type="file" :accept="mimeList" @change="onFileInputChange" />
						<!-- Dummy Box -->
						<label for="file-upload" class="file-upload-box">
							<svg viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
								<rect
									width="20"
									height="20"
									stroke="currentColor"
									stroke-dasharray="1"
									fill="transparent"
								></rect>
							</svg>
						</label>
						<a class="accepted-formats" @click="formatsViewerOpen = !formatsViewerOpen">{{
							t("emote.upload.accepted_formats")
						}}</a>
					</div>
					<!-- Format Matrix -->
					<transition name="matrix">
						<div v-if="formatsViewerOpen" class="matrix">
							<div class="header">{{ t("emote.upload.filetype") }}</div>
							<div class="header">{{ t("emote.upload.animation") }}</div>
							<div class="header">{{ t("emote.upload.transparency") }}</div>
							<template v-for="f of acceptableFileTypes" :key="f.label">
								<div>{{ f.label }}</div>
								<div class="icon">
									<font-awesome-icon v-if="f.animation" :icon="['far', 'check']" color="lime" />
									<font-awesome-icon v-else :icon="['far', 'times']" color="red" />
								</div>
								<div class="icon">
									<font-awesome-icon
										v-if="f.transparency === 'full'"
										:icon="['far', 'check']"
										color="lime"
									/>
									<Tooltip
										v-else-if="f.transparency === 'half'"
										:text="t('emote.upload.half_transparency_tooltip')"
										position="top-end"
									>
										<font-awesome-icon :icon="['far', 'minus']" color="orange" />
									</Tooltip>
									<font-awesome-icon v-else :icon="['far', 'times']" color="red" />
								</div>
							</template>
						</div>
					</transition>
				</div>
			</div>
		</div>
	</div>
</template>

<script setup lang="ts">
import { computed, onMounted, reactive, ref } from "vue";
import { useI18n } from "vue-i18n";
import { LocalStorageKeys } from "@store/lskeys";
import { Emote } from "@structures/Emote";
import { ImageFormat } from "@structures/Common";
import { useQuery } from "@vue/apollo-composable";
import { GetEmote } from "@gql/emotes/emote";
import { useRoute, useRouter } from "vue-router";
import { onClickOutside } from "@vueuse/core";
import TextInput from "@components/form/TextInput.vue";
import Tooltip from "@components/utility/Tooltip.vue";
import Checkbox from "@/components/form/Checkbox.vue";
import EmoteTagList from "./EmoteTagList.vue";
import { useActorStore } from "@store/actor";

const { t } = useI18n();

const props = defineProps<{
	parentID?: string;
	parentData?: string;
}>();
// File Formats
const acceptableFileTypes = [
	{ mime: "image/avif", label: "AVIF", transparency: "full", animation: true },
	{ mime: "image/webp", label: "WEBP", transparency: "full", animation: true },
	{ mime: "image/gif", label: "GIF", transparency: "half", animation: true },
	{ mime: "image/png,image/apng", label: "PNG", transparency: "full", animation: true },
	{ mime: "image/tiff", label: "TIFF", transparency: "full" },
	{ mime: "image/jpeg", label: "JPEG" },
	{ mime: "video/webm", label: "WEBM", animation: true },
	{ mime: "video/mp4", label: "MP4", animation: true },
	{ mime: "video/x-flv", label: "FLV", animation: true },
	{ mime: "video/avi,video/x-msvideo", label: "AVI", animation: true },
	{ mime: "video/quicktime", label: "MOV", transparency: "full", animation: true },
] as FileType[];
const mimeList = acceptableFileTypes.map((ft) => ft.mime).join(",");

// Gather versioning info
const router = useRouter();
const route = useRoute();
const parentID = ref(props.parentID ?? route.query.parentID?.toString() ?? null);
const parentEmote = ref<Emote | null>(props.parentData ? JSON.parse(props.parentData) : null);
if (parentID.value) {
	const { onResult } = useQuery<GetEmote>(GetEmote, { id: parentID.value });
	onResult((res) => (parentEmote.value = res.data.emote));
}

// Formats viewer
const formatsViewerOpen = ref(false);
const formatsViewer = ref<HTMLElement | null>(null);

onMounted(() => {
	onClickOutside(formatsViewer, () => (formatsViewerOpen.value = false));
});

// User info for chat preview
const actor = useActorStore();
const userColor = computed(() => actor.user?.tag_color ?? 0);
const userName = computed(() => actor.user?.username ?? "nani");

// Form
const form = reactive({
	name: "",
	version_description: "",
	zero_width: false,
	private: false,
	tags: [] as string[],
	isCreator: false,

	credits: {
		original_creator: "",
	},
});

// Input File
const imgUrl = ref<string | null>(null);
const aspectRef = ref<HTMLImageElement | null>(null);
const buf = ref<ArrayBuffer | null>(null);
let mime = "";

const onFileInputChange = (event: Event) => {
	const target = event.target as HTMLInputElement;
	const file = target.files?.[0] as File;
	handleFile(file);
};
const dragOverRc = ref(0);
const dragOver = computed(() => !!dragOverRc.value);
const onDropFile = (event: DragEvent) => {
	dragOverRc.value = 0;
	imageAspectRatio.value = 1;
	const file = event.dataTransfer?.files[0] as File;
	if (!file) {
		throw new Error("No file provided during drop event");
	}
	handleFile(file);
};
const onDragEnter = () => {
	dragOverRc.value++;
};
const onDragLeave = () => {
	dragOverRc.value--;
};

const handleFile = async (file: File) => {
	imgUrl.value = URL.createObjectURL(file);

	mime = file.type;
	buf.value = await file.arrayBuffer();
	if (!form.name) {
		form.name = file.name.slice(0, file.name.lastIndexOf("."));
	}
};

const imageAspectRatio = ref(1);
const updateImageAspectRatio = () => {
	if (aspectRef.value === null) {
		imageAspectRatio.value = 1;
	} else {
		const bb = aspectRef.value.getBoundingClientRect();
		imageAspectRatio.value = bb.width / (bb.height || 1);
	}
};

// Upload (network request)
const uploadProgress = ref(0);
const uploadError = ref("");
const upload = () => {
	uploadError.value = "";
	const data = {
		name: !parentEmote.value ? form.name : parentEmote.value.name,
		flags: (form.zero_width ? Emote.Flags.ZERO_WIDTH : 0) | (form.private ? Emote.Flags.PRIVATE : 0),
		tags: form.tags,
	} as Record<string, unknown>;
	// Add versioning data
	if (parentEmote.value) {
		data.parent_id = parentEmote.value.id;
		data.description = form.version_description;
	}
	if (!buf.value) {
		uploadError.value = "Missing file";
		return;
	}

	const req = new XMLHttpRequest();
	req.open("POST", `${import.meta.env.VITE_APP_API_REST as string}/emotes`, true);
	req.setRequestHeader("X-Emote-Data", JSON.stringify(data));
	req.setRequestHeader("Content-Type", mime);
	req.setRequestHeader("Content-Length", buf.value.byteLength.toString(10));
	req.setRequestHeader("Authorization", `Bearer ${localStorage.getItem(LocalStorageKeys.TOKEN)}`);
	req.upload.onprogress = (progress) => (uploadProgress.value = progress.loaded / progress.total);
	req.onload = () => {
		uploadProgress.value = 0;
		if (req.status !== 201) {
			const { error } = JSON.parse(req.responseText);
			uploadError.value = `${error} (${req.status} ${req.statusText})`;
		}
		// upload is complete, redirect to the emote's page
		const { id } = JSON.parse(req.responseText);
		if (typeof id === "string" && id.length > 0) {
			router.push(`/emotes/${id}`);
		}
	};

	req.send(buf.value);
};

const clearError = () => {
	uploadProgress.value = 0;
	uploadError.value = "";
};

//
const txt = computed(() =>
	!parentEmote.value
		? {
				submitEmote: "emote.upload.submit_emote",
				emoteDetails: "emote.upload.emote_details",
				inputEmoteName: "emote.upload.emote_name",
		  }
		: {
				submitEmote: "emote.upload.create_emote_version",
				emoteDetails: "emote.upload.version_details",
				inputEmoteName: "emote.upload.version_name",
		  },
);

// const emoteRegexp = /^[-_A-Za-z():0-9]{100}$/;

interface FileType {
	mime: string;
	label: string;
	animation?: boolean;
	transparency?: "full" | "half" | false;
}
</script>

<style lang="scss">
@import "@scss/emote-upload/emote-upload.scss";
</style>
