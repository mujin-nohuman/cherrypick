<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<a v-vibrate="defaultStore.state.vibrateSystem ? 5 : []" :href="to" :class="active ? activeClass : null" @click.prevent.stop="nav" @contextmenu.prevent.stop="onContextmenu">
	<slot></slot>
</a>
</template>

<script lang="ts" setup>
import { computed } from 'vue';
import * as os from '@/os.js';
import copyToClipboard from '@/scripts/copy-to-clipboard.js';
import { url } from '@/config.js';
import { i18n } from '@/i18n.js';
import { useRouter } from '@/router/supplier.js';
import { defaultStore } from '@/store.js';

const props = withDefaults(defineProps<{
	to: string;
	activeClass?: null | string;
	behavior?: null | 'window' | 'browser';
}>(), {
	activeClass: null,
	behavior: null,
});

const router = useRouter();

const active = computed(() => {
	if (props.activeClass == null) return false;
	const resolved = router.resolve(props.to);
	if (resolved == null) return false;
	if (resolved.route.path === router.currentRoute.value.path) return true;
	if (resolved.route.name == null) return false;
	if (router.currentRoute.value.name == null) return false;
	return resolved.route.name === router.currentRoute.value.name;
});

function onContextmenu(ev) {
	const selection = window.getSelection();
	if (selection && selection.toString() !== '') return;
	os.contextMenu([{
		type: 'label',
		text: props.to,
	}, {
		icon: 'ti ti-app-window',
		text: i18n.ts.openInWindow,
		action: () => {
			os.pageWindow(props.to);
		},
	}, {
		icon: 'ti ti-player-eject',
		text: i18n.ts.showInPage,
		action: () => {
			router.push(props.to, 'forcePage');
		},
	}, { type: 'divider' }, {
		icon: 'ti ti-external-link',
		text: i18n.ts.openInNewTab,
		action: () => {
			window.open(props.to, '_blank', 'noopener');
		},
	}, {
		icon: 'ti ti-link',
		text: i18n.ts.copyLink,
		action: () => {
			copyToClipboard(`${url}${props.to}`);
			os.toast(i18n.ts.copiedLink, 'copied');
		},
	}], ev);
}

function openWindow() {
	os.pageWindow(props.to);
}

function nav(ev: MouseEvent) {
	if (props.behavior === 'browser') {
		location.href = props.to;
		return;
	}

	if (props.behavior) {
		if (props.behavior === 'window') {
			return openWindow();
		}
	}

	if (ev.shiftKey) {
		return openWindow();
	}

	router.push(props.to, ev.ctrlKey ? 'forcePage' : null);
}
</script>
