<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<button
	ref="buttonEl"
	v-ripple="canToggle"
	v-vibrate="defaultStore.state.vibrateSystem ? [10, 30, 40] : []"
	class="_button"
	:class="[$style.root, { [$style.reacted]: note.myReaction == reaction, [$style.canToggle]: (canToggle || alternative), [$style.small]: defaultStore.state.reactionsDisplaySize === 'small', [$style.large]: defaultStore.state.reactionsDisplaySize === 'large' }]"
	@click.stop="(ev) => { canToggle || alternative ? toggleReaction(ev) : stealReaction(ev) }"
	@contextmenu.prevent.stop="menu"
>
	<MkReactionIcon :class="defaultStore.state.limitWidthOfReaction ? $style.limitWidth : ''" :reaction="reaction" :emojiUrl="note.reactionEmojis[reaction.substring(1, reaction.length - 1)]" @click.stop="(ev) => { canToggle || alternative ? toggleReaction(ev) : stealReaction(ev) }"/>
	<span :class="$style.count">{{ count }}</span>
</button>
</template>

<script lang="ts" setup>
import { computed, ComputedRef, inject, onMounted, shallowRef, watch } from 'vue';
import * as Misskey from 'cherrypick-js';
import MkCustomEmojiDetailedDialog from './MkCustomEmojiDetailedDialog.vue';
import XDetails from '@/components/MkReactionsViewer.details.vue';
import MkReactionIcon from '@/components/MkReactionIcon.vue';
import * as os from '@/os.js';
import { misskeyApi, misskeyApiGet } from '@/scripts/misskey-api.js';
import { useTooltip } from '@/scripts/use-tooltip.js';
import { $i } from '@/account.js';
import MkReactionEffect from '@/components/MkReactionEffect.vue';
import { claimAchievement } from '@/scripts/achievements.js';
import { defaultStore } from '@/store.js';
import { i18n } from '@/i18n.js';
import * as sound from '@/scripts/sound.js';
import { checkReactionPermissions } from '@/scripts/check-reaction-permissions.js';
import { customEmojis } from '@/custom-emojis.js';
import copyToClipboard from '@/scripts/copy-to-clipboard.js';

const props = defineProps<{
	reaction: string;
	count: number;
	isInitial: boolean;
	note: Misskey.entities.Note;
}>();

const mock = inject<boolean>('mock', false);

const emit = defineEmits<{
	(ev: 'reactionToggled', emoji: string, newCount: number): void;
}>();

const buttonEl = shallowRef<HTMLElement>();

const isCustomEmoji = computed(() => props.reaction.includes(':'));
const emoji = computed(() => isCustomEmoji.value ? customEmojis.value.find(emoji => emoji.name === props.reaction.replace(/:/g, '').replace(/@\./, '')) : null);

const canToggle = computed(() => {
	return !props.reaction.match(/@\w/) && $i
			&& (emoji.value && checkReactionPermissions($i, props.note, emoji.value))
			|| !isCustomEmoji.value;
});
const canGetInfo = computed(() => !props.reaction.match(/@\w/) && props.reaction.includes(':'));

const reactionName = computed(() => {
	const r = props.reaction.replace(':', '');
	return r.slice(0, r.indexOf('@'));
});

const alternative: ComputedRef<string | null> = computed(() => defaultStore.state.reactableRemoteReactionEnabled ? (customEmojis.value.find(it => it.name === reactionName.value)?.name ?? null) : null);

async function toggleReaction(ev: MouseEvent) {
	if (!canToggle.value) {
		chooseAlternative(ev);
		return;
	}

	const oldReaction = props.note.myReaction;
	if (oldReaction) {
		const confirm = await os.confirm({
			type: 'warning',
			text: oldReaction !== props.reaction ? i18n.ts.changeReactionConfirm : i18n.ts.cancelReactionConfirm,
		});
		if (confirm.canceled) return;

		if (oldReaction !== props.reaction) {
			sound.playMisskeySfx('reaction');
		}

		if (mock) {
			emit('reactionToggled', props.reaction, (props.count - 1));
			return;
		}

		misskeyApi('notes/reactions/delete', {
			noteId: props.note.id,
		}).then(() => {
			if (oldReaction !== props.reaction) {
				misskeyApi('notes/reactions/create', {
					noteId: props.note.id,
					reaction: `:${reactionName.value}:`,
				});
			}
		});
	} else {
		sound.playMisskeySfx('reaction');

		if (mock) {
			emit('reactionToggled', props.reaction, (props.count + 1));
			return;
		}

		misskeyApi('notes/reactions/create', {
			noteId: props.note.id,
			reaction: props.reaction,
		});
		if (props.note.text && props.note.text.length > 100 && (Date.now() - new Date(props.note.createdAt).getTime() < 1000 * 3)) {
			claimAchievement('reactWithoutRead');
		}
	}
}

function stealReaction(ev: MouseEvent) {
	if (props.note.user.host && $i && ($i.isAdmin ?? $i.policies.canManageCustomEmojis)) {
		os.popupMenu([{
			type: 'label',
			text: props.reaction,
		}, {
			text: i18n.ts.import,
			icon: 'ti ti-plus',
			action: async () => {
				await os.apiWithDialog('admin/emoji/steal', {
					name: reactionName.value,
					host: props.note.user.host,
				});
			},
		}, {
			text: `${i18n.ts.doReaction} (${i18n.ts.import})`,
			icon: 'ti ti-mood-plus',
			action: async () => {
				await os.apiWithDialog('admin/emoji/steal', {
					name: reactionName.value,
					host: props.note.user.host,
				});

				await misskeyApi('notes/reactions/create', {
					noteId: props.note.id,
					reaction: `:${reactionName.value}:`,
				});
			},
		}], ev.currentTarget ?? ev.target);
	}
}

async function menu(ev) {
	if (!canGetInfo.value) return;

	os.popupMenu([{
		type: 'label',
		text: `:${reactionName.value}:`,
	}, {
		text: i18n.ts.info,
		icon: 'ti ti-info-circle',
		action: async () => {
			os.popup(MkCustomEmojiDetailedDialog, {
				emoji: await misskeyApiGet('emoji', {
					name: props.reaction.replace(/:/g, '').replace(/@\./, ''),
				}),
			});
		},
	}, customEmojis.value.find(it => it.name === reactionName.value)?.name ? {
		text: i18n.ts.copy,
		icon: 'ti ti-copy',
		action: () => {
			copyToClipboard(`:${reactionName.value}:`);
			os.toast(i18n.ts.copied, 'copied');
		},
	} : undefined], ev.currentTarget ?? ev.target);
}

function anime() {
	if (document.hidden || !defaultStore.state.animation || buttonEl.value == null) return;

	const rect = buttonEl.value.getBoundingClientRect();
	const x = rect.left + 16;
	const y = rect.top + (buttonEl.value.offsetHeight / 2);
	os.popup(MkReactionEffect, { reaction: props.reaction, x, y }, {}, 'end');
}

function chooseAlternative(ev) {
	// メニュー表示にして、モデレーター以上の場合は登録もできるように
	if (!alternative.value) return;
	console.log(alternative.value);
	misskeyApi('notes/reactions/create', {
		noteId: props.note.id,
		reaction: `:${alternative.value}:`,
	});
}

watch(() => props.count, (newCount, oldCount) => {
	if (oldCount < newCount) anime();
});

onMounted(() => {
	if (!props.isInitial) anime();
});

if (!mock) {
	useTooltip(buttonEl, async (showing) => {
		const reactions = await misskeyApiGet('notes/reactions', {
			noteId: props.note.id,
			type: props.reaction,
			limit: 10,
			_cacheKey_: props.count,
		});

		const users = reactions.map(x => x.user);

		os.popup(XDetails, {
			showing,
			reaction: props.reaction,
			users,
			count: props.count,
			targetElement: buttonEl.value,
		}, {}, 'closed');
	}, 100);
}
</script>

<style lang="scss" module>
.root {
	display: inline-flex;
	height: 38px;
	margin: 2px;
	padding: 0 12px;
	font-size: 1.35em;
	border-radius: 999px;
	align-items: center;
	justify-content: center;

	&.canToggle {
		background: var(--buttonBg);

		&:hover {
			background: var(--buttonHoverBg, rgba(0, 0, 0, 0.1));
		}
	}

	&:not(.canToggle) {
		cursor: default;
	}

	&.small {
		height: 30px;
		font-size: 1em;

		> .count {
			font-size: 0.9em;
			line-height: 22px;
		}
	}

	&.large {
		height: 46px;
		font-size: 1.8em;
		padding: 4px 16px;

		> .count {
			font-size: 0.6em;
			line-height: 50px;
			margin: 0 0 0 8px;
		}
	}

	&.reacted, &.reacted:hover {
		background: var(--accentedBg);
		color: var(--accent);
		box-shadow: 0 0 0 1px var(--accent) inset;

		> .count {
			color: var(--accent);
		}

		> .icon {
			filter: drop-shadow(0 0 2px rgba(0, 0, 0, 0.5));
		}
	}
}

.limitWidth {
	max-width: 70px;
	object-fit: contain;
}

.count {
	font-size: 0.9em;
	line-height: 32px;
	margin: 0 0 0 5px;
}
</style>
