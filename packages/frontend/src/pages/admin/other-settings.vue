<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<MkStickyContainer>
	<template #header><XHeader :actions="headerActions" :tabs="headerTabs"/></template>
	<MkSpacer :contentMax="700" :marginMin="16" :marginMax="32">
		<FormSuspense :p="init">
			<div class="_gaps">
				<FormLink to="/admin/update"><template #icon><i class="ti ti-refresh"></i></template>{{ i18n.ts.cherrypickUpdate }}</FormLink>

				<div class="_panel" style="padding: 16px;">
					<MkSwitch v-model="enableServerMachineStats">
						<template #label>{{ i18n.ts.enableServerMachineStats }}</template>
						<template #caption>{{ i18n.ts.turnOffToImprovePerformance }}</template>
					</MkSwitch>
				</div>

				<div class="_panel" style="padding: 16px;">
					<MkSwitch v-model="enableIdenticonGeneration">
						<template #label>{{ i18n.ts.enableIdenticonGeneration }}</template>
						<template #caption>{{ i18n.ts.turnOffToImprovePerformance }}</template>
					</MkSwitch>
				</div>

				<div class="_panel" style="padding: 16px;">
					<MkSwitch v-model="enableChartsForRemoteUser">
						<template #label>{{ i18n.ts.enableChartsForRemoteUser }}</template>
						<template #caption>{{ i18n.ts.turnOffToImprovePerformance }}</template>
					</MkSwitch>
				</div>

				<div class="_panel" style="padding: 16px;">
					<MkSwitch v-model="enableChartsForFederatedInstances">
						<template #label>{{ i18n.ts.enableChartsForFederatedInstances }}</template>
						<template #caption>{{ i18n.ts.turnOffToImprovePerformance }}</template>
					</MkSwitch>
				</div>

				<div class="_panel" style="padding: 16px;">
					<MkSwitch v-model="doNotSendNotificationEmailsForAbuseReport">
						<template #label>{{ i18n.ts.doNotSendNotificationEmailsForAbuseReport }}</template>
					</MkSwitch>
				</div>
			</div>
		</FormSuspense>
	</MkSpacer>
</MkStickyContainer>
</template>

<script lang="ts" setup>
import { ref, computed } from 'vue';
import XHeader from './_header_.vue';
import FormSuspense from '@/components/form/suspense.vue';
import * as os from '@/os.js';
import { misskeyApi } from '@/scripts/misskey-api.js';
import { fetchInstance } from '@/instance.js';
import { i18n } from '@/i18n.js';
import { definePageMetadata } from '@/scripts/page-metadata.js';
import MkSwitch from '@/components/MkSwitch.vue';
import FormLink from '@/components/form/link.vue';

const enableServerMachineStats = ref<boolean>(false);
const enableIdenticonGeneration = ref<boolean>(false);
const enableChartsForRemoteUser = ref<boolean>(false);
const enableChartsForFederatedInstances = ref<boolean>(false);
const doNotSendNotificationEmailsForAbuseReport = ref<boolean>(false);

async function init() {
	const meta = await misskeyApi('admin/meta');
	enableServerMachineStats.value = meta.enableServerMachineStats;
	enableIdenticonGeneration.value = meta.enableIdenticonGeneration;
	enableChartsForRemoteUser.value = meta.enableChartsForRemoteUser;
	enableChartsForFederatedInstances.value = meta.enableChartsForFederatedInstances;
	doNotSendNotificationEmailsForAbuseReport.value = meta.doNotSendNotificationEmailsForAbuseReport;
}

function save() {
	os.apiWithDialog('admin/update-meta', {
		enableServerMachineStats: enableServerMachineStats.value,
		enableIdenticonGeneration: enableIdenticonGeneration.value,
		enableChartsForRemoteUser: enableChartsForRemoteUser.value,
		enableChartsForFederatedInstances: enableChartsForFederatedInstances.value,
		doNotSendNotificationEmailsForAbuseReport: doNotSendNotificationEmailsForAbuseReport.value,
	}).then(() => {
		fetchInstance();
	});
}

const headerActions = computed(() => [{
	asFullButton: true,
	icon: 'ti ti-check',
	text: i18n.ts.save,
	handler: save,
}]);

const headerTabs = computed(() => []);

definePageMetadata(() => ({
	title: i18n.ts.other,
	icon: 'ti ti-adjustments',
}));
</script>
