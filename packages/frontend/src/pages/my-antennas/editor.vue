<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<MkStickyContainer>
	<template #header><MkPageHeader :actions="headerActions" :tabs="headerTabs"/></template>
	<MkSpacer :contentMax="700">
		<div>
			<div class="_gaps_m">
				<MkInput v-model="name">
					<template #label>{{ i18n.ts.name }}</template>
				</MkInput>
				<MkSelect v-model="src">
					<template #label>{{ i18n.ts.antennaSource }}</template>
					<option value="all">{{ i18n.ts._antennaSources.all }}</option>
					<!--<option value="home">{{ i18n.ts._antennaSources.homeTimeline }}</option>-->
					<option value="users">{{ i18n.ts._antennaSources.users }}</option>
					<option value="list">{{ i18n.ts._antennaSources.userList }}</option>
					<option value="group">{{ i18n.ts._antennaSources.userGroup }}</option>
					<option value="users_blacklist">{{ i18n.ts._antennaSources.userBlacklist }}</option>
				</MkSelect>
				<MkSelect v-if="src === 'list'" v-model="userListId">
					<template #label>{{ i18n.ts.userList }}</template>
					<option v-for="list in userLists" :key="list.id" :value="list.id">{{ list.name }}</option>
				</MkSelect>
				<MkSelect v-else-if="src === 'group'" v-model="userGroupId">
					<template #label>{{ i18n.ts.group }}</template>
					<option v-for="group in userGroups" :key="group.id" :value="group.id">{{ group.name }}</option>
				</MkSelect>
				<MkTextarea v-else-if="src === 'users' || src === 'users_blacklist'" v-model="users">
					<template #label>{{ i18n.ts.users }}</template>
					<template #caption>{{ i18n.ts.antennaUsersDescription }} <button class="_textButton" @click="addUser">{{ i18n.ts.addUser }}</button></template>
				</MkTextarea>
				<MkSwitch v-model="withReplies">{{ i18n.ts.withReplies }}</MkSwitch>
				<MkTextarea v-model="keywords">
					<template #label>{{ i18n.ts.antennaKeywords }}</template>
					<template #caption>{{ i18n.ts.antennaKeywordsDescription }}</template>
				</MkTextarea>
				<MkTextarea v-model="excludeKeywords">
					<template #label>{{ i18n.ts.antennaExcludeKeywords }}</template>
					<template #caption>{{ i18n.ts.antennaKeywordsDescription }}</template>
				</MkTextarea>
				<MkSwitch v-model="localOnly">{{ i18n.ts.localOnly }}</MkSwitch>
				<MkSwitch v-model="caseSensitive">{{ i18n.ts.caseSensitive }}</MkSwitch>
				<MkSwitch v-model="withFile">{{ i18n.ts.withFileAntenna }}</MkSwitch>
				<MkSwitch v-model="notify">{{ i18n.ts.notifyAntenna }}</MkSwitch>
			</div>
			<div :class="$style.actions">
				<MkButton inline primary @click="saveAntenna()"><i class="ti ti-device-floppy"></i> {{ i18n.ts.save }}</MkButton>
				<MkButton v-if="antenna.id != null" inline danger @click="deleteAntenna()"><i class="ti ti-trash"></i> {{ i18n.ts.delete }}</MkButton>
			</div>
		</div>
	</MkSpacer>
</MkStickyContainer>
</template>

<script lang="ts" setup>
import { watch, ref, computed } from 'vue';
import * as Misskey from 'cherrypick-js';
import MkButton from '@/components/MkButton.vue';
import MkInput from '@/components/MkInput.vue';
import MkTextarea from '@/components/MkTextarea.vue';
import MkSelect from '@/components/MkSelect.vue';
import MkSwitch from '@/components/MkSwitch.vue';
import * as os from '@/os.js';
import { misskeyApi } from '@/scripts/misskey-api.js';
import { i18n } from '@/i18n.js';

const props = defineProps<{
	antenna: Misskey.entities.Antenna
}>();

const emit = defineEmits<{
	(ev: 'created'): void,
	(ev: 'updated'): void,
	(ev: 'deleted'): void,
}>();

const name = ref<string>(props.antenna.name);
const src = ref<Misskey.entities.AntennasCreateRequest['src']>(props.antenna.src);
const userListId = ref<string | null>(props.antenna.userListId);
const userGroupId = ref<string | null>(props.antenna.userGroupId);
const users = ref<string>(props.antenna.users.join('\n'));
const keywords = ref<string>(props.antenna.keywords.map(x => x.join(' ')).join('\n'));
const excludeKeywords = ref<string>(props.antenna.excludeKeywords.map(x => x.join(' ')).join('\n'));
const caseSensitive = ref<boolean>(props.antenna.caseSensitive);
const localOnly = ref<boolean>(props.antenna.localOnly);
const withReplies = ref<boolean>(props.antenna.withReplies);
const withFile = ref<boolean>(props.antenna.withFile);
const notify = ref<boolean>(props.antenna.notify);
const userLists = ref<Misskey.entities.UserList[] | null>(null);
const userGroups = ref<Misskey.entities.UserGroup[] | null>(null);

watch(() => src.value, async () => {
	if (src.value === 'list' && userLists.value === null) {
		userLists.value = await misskeyApi('users/lists/list');
	}

	if (src.value === 'group' && userGroups.value === null) {
		const groups1 = await misskeyApi('users/groups/owned');
		const groups2 = await misskeyApi('users/groups/joined');

		userGroups.value = [...groups1, ...groups2];
	}
});

const headerActions = computed(() => []);

const headerTabs = computed(() => []);

async function saveAntenna() {
	const antennaData = {
		name: name.value,
		src: src.value,
		userListId: userListId.value,
		userGroupId: userGroupId.value,
		withReplies: withReplies.value,
		withFile: withFile.value,
		notify: notify.value,
		caseSensitive: caseSensitive.value,
		localOnly: localOnly.value,
		users: users.value.trim().split('\n').map(x => x.trim()),
		keywords: keywords.value.trim().split('\n').map(x => x.trim().split(' ')),
		excludeKeywords: excludeKeywords.value.trim().split('\n').map(x => x.trim().split(' ')),
	};

	if (props.antenna.id == null) {
		await os.apiWithDialog('antennas/create', antennaData);
		emit('created');
	} else {
		await os.apiWithDialog('antennas/update', { ...antennaData, antennaId: props.antenna.id });
		emit('updated');
	}
}

async function deleteAntenna() {
	const { canceled } = await os.confirm({
		type: 'warning',
		text: i18n.tsx.removeAreYouSure({ x: props.antenna.name }),
	});
	if (canceled) return;

	await misskeyApi('antennas/delete', {
		antennaId: props.antenna.id,
	});

	os.success();
	emit('deleted');
}

function addUser() {
	os.selectUser({ includeSelf: true }).then(user => {
		users.value = users.value.trim();
		users.value += '\n@' + Misskey.acct.toString(user as any);
		users.value = users.value.trim();
	});
}
</script>

<style lang="scss" module>
.actions {
	margin-top: 16px;
	padding: 24px 0;
	border-top: solid 0.5px var(--divider);
}
</style>
