<script lang="ts" setup>
import { useCanvasOperations } from '@/composables/useCanvasOperations';
import { useI18n } from '@/composables/useI18n';
import { useRunWorkflow } from '@/composables/useRunWorkflow';
import { CHAT_TRIGGER_NODE_TYPE, LOCAL_STORAGE_CANVAS_TRIGGER_BUTTON_VARIANT } from '@/constants';
import { useUIStore } from '@/stores/ui.store';
import { useWorkflowsStore } from '@/stores/workflows.store';
import { useLocalStorage } from '@vueuse/core';
import { computed, useCssModule } from 'vue';
import { useRouter } from 'vue-router';

const {
	name,
	type,
	hovered,
	disabled,
	class: cls,
} = defineProps<{
	name: string;
	type: string;
	hovered?: boolean;
	disabled?: boolean;
	class?: string;
}>();

const variant = useLocalStorage<1 | 2>(LOCAL_STORAGE_CANVAS_TRIGGER_BUTTON_VARIANT, 2);

const style = useCssModule();
const containerClass = computed(() => ({
	[cls ?? '']: true,
	[style.container]: true,
	[style.interactive]: !disabled,
	[style.hovered]: !!hovered,
	[style.variant1]: variant.value === 1,
	[style.variant2]: variant.value === 2,
}));

const router = useRouter();
const i18n = useI18n();
const workflowsStore = useWorkflowsStore();
const uiStore = useUIStore();
const { runEntireWorkflow } = useRunWorkflow({ router });
const { toggleChatOpen } = useCanvasOperations({ router });

const isChatOpen = computed(() => workflowsStore.isChatPanelOpen);
const isExecuting = computed(() => uiStore.isActionActive.workflowRunning);
const testId = computed(() => `execute-workflow-button-${name}`);
</script>

<template>
	<!-- click and mousedown event are suppressed to avoid unwanted selection or dragging of the node -->
	<div :class="containerClass" @click.stop.prevent @mousedown.stop.prevent>
		<div>
			<div v-if="variant === 2" :class="$style.bolt">
				<FontAwesomeIcon icon="bolt" size="lg" />
			</div>

			<N8nButton
				v-if="variant === 1 && type === CHAT_TRIGGER_NODE_TYPE"
				type="secondary"
				size="large"
				:disabled="isExecuting"
				:data-test-id="testId"
				@click.capture="toggleChatOpen('node')"
				>{{ isChatOpen ? i18n.baseText('chat.hide') : i18n.baseText('chat.open') }}</N8nButton
			>
			<N8nButton
				v-else-if="variant === 1"
				type="secondary"
				size="large"
				:disabled="isExecuting"
				:data-test-id="testId"
				@click.capture="runEntireWorkflow('node', name)"
				>{{ i18n.baseText('nodeView.runButtonText.executeWorkflow') }}</N8nButton
			>
			<N8nButton
				v-else-if="variant === 2 && type === CHAT_TRIGGER_NODE_TYPE"
				:type="isChatOpen ? 'secondary' : 'primary'"
				size="large"
				:disabled="isExecuting"
				:data-test-id="testId"
				:label="isChatOpen ? i18n.baseText('chat.hide') : i18n.baseText('chat.open')"
				@click.capture="toggleChatOpen('node')"
			/>
			<N8nButton
				v-else
				type="primary"
				size="large"
				:disabled="isExecuting"
				:data-test-id="testId"
				:label="i18n.baseText('nodeView.runButtonText.executeWorkflow')"
				@click.capture="runEntireWorkflow('node', name)"
			/>
		</div>
	</div>
</template>

<style lang="scss" module>
.container {
	z-index: -1;
	position: absolute;
	display: flex;
	align-items: center;
	height: 100%;
	right: 100%;
	top: 0;
	pointer-events: none;

	& > div {
		position: relative;
		display: flex;
		align-items: center;
	}

	&.hovered button {
		pointer-events: all;
	}
}

.variant1 {
	& button {
		margin-right: var(--spacing-xl);
		display: none;
	}

	&.interactive button {
		display: block;
	}
}

.variant2 {
	& button {
		margin-right: var(--spacing-s);
		opacity: 0;
		translate: -12px 0;
		transition:
			translate 0.1s ease-in,
			opacity 0.1s ease-in;
	}

	&.interactive.hovered button {
		opacity: 1;
		translate: 0 0;
	}
}

.bolt {
	position: absolute;
	right: 0;
	color: var(--color-primary);
	padding: var(--spacing-s);
	opacity: 1;
	translate: 0 0;
	transition:
		translate 0.1s ease-in,
		opacity 0.1s ease-in;

	.container.interactive.hovered & {
		translate: -12px 0;
		opacity: 0;
	}
}
</style>
