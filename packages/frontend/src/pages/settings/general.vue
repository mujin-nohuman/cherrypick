<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<div class="_gaps_m">
	<MkSelect v-model="lang">
		<template #label>{{ i18n.ts.uiLanguage }}</template>
		<option v-for="x in langs" :key="x[0]" :value="x[0]">{{ x[1] }}</option>
		<template #caption>
			<I18n :src="i18n.ts.i18nInfo" tag="span">
				<template #link>
					<MkLink url="https://crowdin.com/project/misskey">Crowdin</MkLink>
				</template>
			</I18n>
		</template>
	</MkSelect>

	<MkRadios v-model="hemisphere">
		<template #label>{{ i18n.ts.hemisphere }}</template>
		<option value="N">{{ i18n.ts._hemisphere.N }}</option>
		<option value="S">{{ i18n.ts._hemisphere.S }}</option>
		<template #caption>{{ i18n.ts._hemisphere.caption }}</template>
	</MkRadios>

	<MkRadios v-model="overridedDeviceKind">
		<template #label>{{ i18n.ts.overridedDeviceKind }}</template>
		<option :value="null">{{ i18n.ts.auto }}</option>
		<option value="smartphone"><i class="ti ti-device-mobile"/> {{ i18n.ts.smartphone }}</option>
		<option value="tablet"><i class="ti ti-device-tablet"/> {{ i18n.ts.tablet }}</option>
		<option value="desktop"><i class="ti ti-device-desktop"/> {{ i18n.ts.desktop }}</option>
	</MkRadios>

	<FormSection>
		<div class="_gaps_s">
			<MkSwitch v-model="showFixedPostForm">{{ i18n.ts.showFixedPostForm }}</MkSwitch>
			<MkSwitch v-model="showFixedPostFormInChannel">{{ i18n.ts.showFixedPostFormInChannel }}</MkSwitch>
			<MkFolder>
				<template #label>{{ i18n.ts.pinnedList }}</template>
				<!-- 複数ピン止め管理できるようにしたいけどめんどいので一旦ひとつのみ -->
				<MkButton v-if="defaultStore.reactiveState.pinnedUserLists.value.length === 0" @click="setPinnedList()">{{ i18n.ts.add }}</MkButton>
				<MkButton v-else danger @click="removePinnedList()"><i class="ti ti-trash"></i> {{ i18n.ts.remove }}</MkButton>
			</MkFolder>
		</div>
	</FormSection>

	<FormSection>
		<template #label>{{ i18n.ts.displayOfNote }}</template>

		<div class="_gaps_m">
			<div class="_gaps_s">
				<MkSwitch v-model="showNoteActionsOnlyHover">{{ i18n.ts.showNoteActionsOnlyHover }}</MkSwitch>
				<MkSwitch v-model="showClipButtonInNoteFooter">{{ i18n.ts.showClipButtonInNoteFooter }}</MkSwitch>
				<MkSwitch v-model="showTranslateButtonInNote">{{ i18n.ts.showTranslateButtonInNote }} <span class="_beta">CherryPick</span></MkSwitch>
				<MkSwitch v-model="collapseRenotes">{{ i18n.ts.collapseRenotes }}</MkSwitch>
				<MkSwitch v-model="collapseDefault">{{ i18n.ts.collapseDefault }} <span class="_beta">CherryPick</span></MkSwitch>
				<MkSwitch v-model="advancedMfm">{{ i18n.ts.enableAdvancedMfm }}</MkSwitch>
				<MkSwitch v-if="advancedMfm" v-model="animatedMfm">{{ i18n.ts.enableAnimatedMfm }}</MkSwitch>
				<div :class="$style.mfmPreview" class="_panel">
					<div v-if="advancedMfm && animatedMfm" style="margin: 0 0 8px; font-size: 1.5em;">
						<Mfm :key="emojiStyle" text="$[jelly 🍮] $[spin 🍪] $[shake 🍭]"/>
					</div>
					<div v-else style="margin: 0 0 8px; font-size: 1.5em;">
						<Mfm :key="emojiStyle" text="🍮 🍪 🍭"/>
					</div>
				</div>
				<MkSwitch v-if="advancedMfm" v-model="enableQuickAddMfmFunction">{{ i18n.ts.enableQuickAddMfmFunction }}</MkSwitch>
				<MkSwitch v-model="showGapBetweenNotesInTimeline">{{ i18n.ts.showGapBetweenNotesInTimeline }}</MkSwitch>
				<MkSwitch v-model="loadRawImages">{{ i18n.ts.loadRawImages }}</MkSwitch>
				<MkRadios v-model="reactionsDisplaySize">
					<template #label>{{ i18n.ts.reactionsDisplaySize }}</template>
					<option value="small">{{ i18n.ts.small }}</option>
					<option value="medium">{{ i18n.ts.medium }}</option>
					<option value="large">{{ i18n.ts.large }}</option>
				</MkRadios>
				<MkSwitch v-model="limitWidthOfReaction">{{ i18n.ts.limitWidthOfReaction }}</MkSwitch>
				<MkSwitch v-model="hideAvatarsInNote">{{ i18n.ts.hideAvatarsInNote }} <span class="_beta">CherryPick</span></MkSwitch>
				<MkSwitch v-model="enableAbsoluteTime">{{ i18n.ts.enableAbsoluteTime }} <span class="_beta">CherryPick</span></MkSwitch>
				<MkSwitch v-model="enableMarkByDate" :disabled="defaultStore.state.enableAbsoluteTime">{{ i18n.ts.enableMarkByDate }} <span class="_beta">CherryPick</span></MkSwitch>
				<MkSwitch v-model="showSubNoteFooterButton">{{ i18n.ts.showSubNoteFooterButton }}<template #caption>{{ i18n.ts.showSubNoteFooterButtonDescription }}</template> <span class="_beta">CherryPick</span></MkSwitch>
				<MkSwitch v-model="infoButtonForNoteActionsEnabled">{{ i18n.ts.infoButtonForNoteActions }}<template #caption>{{ i18n.ts.infoButtonForNoteActionsDescription }}</template> <span class="_beta">CherryPick</span></MkSwitch>
				<MkSwitch v-model="showReplyInNotification">{{ i18n.ts.showReplyInNotification }} <span class="_beta">CherryPick</span></MkSwitch>
				<MkSwitch v-model="renoteQuoteButtonSeparation">{{ i18n.ts.renoteQuoteButtonSeparation }} <span class="_beta">CherryPick</span></MkSwitch>
				<MkSwitch v-model="renoteVisibilitySelection">{{ i18n.ts.showRenoteVisibilitySelector }} <span class="_beta">CherryPick</span></MkSwitch>
				<MkSelect v-if="!renoteVisibilitySelection" v-model="forceRenoteVisibilitySelection">
					<template #label>{{ i18n.ts.forceRenoteVisibilitySelector }}</template>
					<option value="none">{{ i18n.ts.none }}</option>
					<option value="public">{{ i18n.ts._visibility.public }}</option>
					<option value="home">{{ i18n.ts._visibility.home }}</option>
					<option value="followers">{{ i18n.ts._visibility.followers }}</option>
				</MkSelect>
				<MkSwitch v-model="showFixedPostFormInReplies">{{ i18n.ts.showFixedPostFormInReplies }}<template #caption>{{ i18n.ts.showFixedPostFormInRepliesDescription }}</template> <span class="_beta">CherryPick</span></MkSwitch>
				<MkSwitch v-model="allMediaNoteCollapse">{{ i18n.ts.allMediaNoteCollapse }} <span class="_beta">CherryPick</span></MkSwitch>
			</div>

			<MkSelect v-model="instanceTicker">
				<template #label>{{ i18n.ts.instanceTicker }}</template>
				<option value="always">{{ i18n.ts._instanceTicker.always }}</option>
				<option value="remote">{{ i18n.ts._instanceTicker.remote }}</option>
				<option value="none">{{ i18n.ts._instanceTicker.none }}</option>
			</MkSelect>

			<MkSelect v-model="nsfw">
				<template #label>{{ i18n.ts.displayOfSensitiveMedia }}</template>
				<option value="respect">{{ i18n.ts._displayOfSensitiveMedia.respect }}</option>
				<option value="ignore">{{ i18n.ts._displayOfSensitiveMedia.ignore }}</option>
				<option value="force">{{ i18n.ts._displayOfSensitiveMedia.force }}</option>
			</MkSelect>

			<MkSelect v-model="nsfwOpenBehavior">
				<template #label>{{ i18n.ts.nsfwOpenBehavior }} <span class="_beta">CherryPick</span></template>
				<option value="click">{{ i18n.ts._nsfwOpenBehavior.click }}</option>
				<option value="doubleClick">{{ i18n.ts._nsfwOpenBehavior.doubleClick }}</option>
			</MkSelect>

			<MkRadios v-model="mediaListWithOneImageAppearance">
				<template #label>{{ i18n.ts.mediaListWithOneImageAppearance }}</template>
				<option value="expand">{{ i18n.ts.default }}</option>
				<option value="16_9">{{ i18n.tsx.limitTo({ x: '16:9' }) }}</option>
				<option value="1_1">{{ i18n.tsx.limitTo({ x: '1:1' }) }}</option>
				<option value="2_3">{{ i18n.tsx.limitTo({ x: '2:3' }) }}</option>
			</MkRadios>
		</div>
	</FormSection>

	<FormSection>
		<template #label>{{ i18n.ts.notificationDisplay }}</template>

		<div class="_gaps_m">
			<MkSwitch v-model="useGroupedNotifications">{{ i18n.ts.useGroupedNotifications }}</MkSwitch>

			<MkRadios v-model="notificationPosition">
				<template #label>{{ i18n.ts.position }}</template>
				<option value="leftTop"><i class="ti ti-align-box-left-top"></i> {{ i18n.ts.leftTop }}</option>
				<option value="rightTop"><i class="ti ti-align-box-right-top"></i> {{ i18n.ts.rightTop }}</option>
				<option value="leftBottom"><i class="ti ti-align-box-left-bottom"></i> {{ i18n.ts.leftBottom }}</option>
				<option value="rightBottom"><i class="ti ti-align-box-right-bottom"></i> {{ i18n.ts.rightBottom }}</option>
			</MkRadios>

			<MkRadios v-model="notificationStackAxis">
				<template #label>{{ i18n.ts.stackAxis }}</template>
				<option value="vertical"><i class="ti ti-carousel-vertical"></i> {{ i18n.ts.vertical }}</option>
				<option value="horizontal"><i class="ti ti-carousel-horizontal"></i> {{ i18n.ts.horizontal }}</option>
			</MkRadios>

			<MkButton @click="testNotification">{{ i18n.ts._notification.checkNotificationBehavior }}</MkButton>
		</div>
	</FormSection>

	<FormSection>
		<template #label>{{ i18n.ts.appearance }}</template>

		<div class="_gaps_m">
			<div class="_gaps_s">
				<MkSwitch v-model="reduceAnimation">{{ i18n.ts.reduceUiAnimation }}</MkSwitch>
				<MkSwitch v-model="useBlurEffect">{{ i18n.ts.useBlurEffect }}<template #caption>{{ i18n.ts.useBlurEffectDescription }}</template></MkSwitch>
				<MkSwitch v-model="useBlurEffectForModal">{{ i18n.ts.useBlurEffectForModal }}</MkSwitch>
				<MkSwitch v-if="useBlurEffect && useBlurEffectForModal" v-model="removeModalBgColorForBlur">{{ i18n.ts.removeModalBgColorForBlur }} <span class="_beta">CherryPick</span></MkSwitch>
				<MkSwitch v-model="disableShowingAnimatedImages">{{ i18n.ts.disableShowingAnimatedImages }}<template #caption><i class="ti ti-alert-triangle" style="color: var(--warn);"></i> {{ i18n.ts.disableShowingAnimatedImagesDescription }}</template></MkSwitch>
				<MkSelect v-if="!disableShowingAnimatedImages" v-model="showingAnimatedImages" style="margin-left: 44px;">
					<option value="always">{{ i18n.ts._showingAnimatedImages.always }}</option>
					<option value="interaction">{{ i18n.ts._showingAnimatedImages.interaction }}</option>
					<option value="inactive">{{ i18n.ts._showingAnimatedImages.inactive }}</option>
					<template #caption>{{ i18n.ts.showingAnimatedImagesDescription }}</template>
				</MkSelect>
				<MkSwitch v-model="highlightSensitiveMedia">{{ i18n.ts.highlightSensitiveMedia }}</MkSwitch>
				<MkSwitch v-model="squareAvatars">{{ i18n.ts.squareAvatars }}</MkSwitch>
				<MkSwitch v-model="showAvatarDecorations">{{ i18n.ts.showAvatarDecorations }}</MkSwitch>
				<MkSwitch v-model="useSystemFont">{{ i18n.ts.useSystemFont }}</MkSwitch>
				<MkSwitch v-model="disableDrawer">{{ i18n.ts.disableDrawer }}</MkSwitch>
				<MkSwitch v-model="forceShowAds">{{ i18n.ts.forceShowAds }}</MkSwitch>
				<MkSwitch v-model="enableSeasonalScreenEffect">{{ i18n.ts.seasonalScreenEffect }}</MkSwitch>
				<MkSwitch v-model="showUnreadNotificationsCount">{{ i18n.ts.showUnreadNotificationsCount }}</MkSwitch>
			</div>
			<div>
				<MkRadios v-model="emojiStyle">
					<template #label>{{ i18n.ts.emojiStyle }}</template>
					<option value="native">{{ i18n.ts.native }}</option>
					<option value="fluentEmoji">Fluent Emoji</option>
					<option value="twemoji">Twemoji</option>
				</MkRadios>
				<div style="margin: 8px 0 0 0; font-size: 1.5em;"><Mfm :key="emojiStyle" text="🍮🍦🍭🍩🍰🍫🍬🥞🍪"/></div>
			</div>

			<!--
			<MkRadios v-model="fontSize">
				<template #label>{{ i18n.ts.fontSize }}</template>
				<option value="1"><span style="font-size: 12px;">Aa</span></option>
				<option value="2"><span style="font-size: 13px;">Aa</span></option>
				<option :value="null"><span style="font-size: 14px;">Aa</span></option>
				<option value="3"><span style="font-size: 15px;">Aa</span></option>
				<option value="4"><span style="font-size: 16px;">Aa</span></option>
				<option value="5"><span style="font-size: 17px;">Aa</span></option>
				<option value="6"><span style="font-size: 18px;">Aa</span></option>
				<option value="7"><span style="font-size: 19px;">Aa</span></option>
				<option value="8"><span style="font-size: 20px;">Aa</span></option>
			</MkRadios>
			-->

			<div>
				<div :class="$style.label">{{ i18n.ts.fontSize }} <span class="_beta">CherryPick</span></div>
				<div :class="$style.fontSize" class="_panel">
					<div v-if="fontSize === 1" style="font-size: 7px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 2" style="font-size: 8px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 3" style="font-size: 9px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 4" style="font-size: 10px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 5" style="font-size: 11px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 6" style="font-size: 12px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 7" style="font-size: 13px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 8" style="font-size: 14px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 9" style="font-size: 15px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 10" style="font-size: 16px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 11" style="font-size: 17px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 12" style="font-size: 18px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 13" style="font-size: 19px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 14" style="font-size: 20px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 15" style="font-size: 21px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 16" style="font-size: 22px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 17" style="font-size: 23px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 18" style="font-size: 24px;">{{ i18n.ts._mfm.dummy }}</div>
					<div v-else-if="fontSize === 19" style="font-size: 25px;">{{ i18n.ts._mfm.dummy }}</div>
				</div>
				<div :class="$style.fontSizeSlider">
					<div :class="$style.fontSizeLeft">Aa</div>
					<MkRange
						v-model="fontSize" style="position: initial !important; width: 100%; margin: 0 -10px;" :min="1" :max="19" :step="1" easing :textConverter="(v) =>
							v === 1 ? '7px' :
							v === 2 ? '8px' :
							v === 3 ? '9px' :
							v === 4 ? '10px' :
							v === 5 ? '11px' :
							v === 6 ? '12px' :
							v === 7 ? '13px' :
							v === 8 ? '14px' :
							v === 9 ? '15px' :
							v === 10 ? '16px' :
							v === 11 ? '17px' :
							v === 12 ? '18px' :
							v === 13 ? '19px' :
							v === 14 ? '20px' :
							v === 15 ? '21px' :
							v === 16 ? '22px' :
							v === 17 ? '23px' :
							v === 18 ? '24px' :
							v === 19 ? '25px' : ''"
					>
					</MkRange>
					<div :class="$style.fontSizeRight">Aa</div>
				</div>
				<MkInfo v-if="fontSize != fontSizeBefore" style="margin-top: 10px;">{{ i18n.ts.reloadToApplySetting2 }}</MkInfo>
				<MkSwitch v-model="useBoldFont" style="margin-top: .75em;">{{ i18n.ts.useBoldFont }}</MkSwitch>
			</div>
		</div>
	</FormSection>

	<FormSection>
		<template #label>{{ i18n.ts.behavior }}</template>

		<div class="_gaps_m">
			<div class="_gaps_s">
				<MkSwitch v-model="imageNewTab">{{ i18n.ts.openImageInNewTab }}</MkSwitch>
				<MkSwitch v-model="useReactionPickerForContextMenu">{{ i18n.ts.useReactionPickerForContextMenu }}</MkSwitch>
				<MkSwitch v-model="enableInfiniteScroll">{{ i18n.ts.enableInfiniteScroll }}</MkSwitch>
				<MkSwitch v-model="keepScreenOn">{{ i18n.ts.keepScreenOn }}</MkSwitch>
				<MkSwitch v-model="disableStreamingTimeline">{{ i18n.ts.disableStreamingTimeline }}</MkSwitch>
				<MkSwitch v-model="enableHorizontalSwipe">{{ i18n.ts.enableHorizontalSwipe }}</MkSwitch>
			</div>
			<MkSelect v-model="serverDisconnectedBehavior">
				<template #label>{{ i18n.ts.whenServerDisconnected }} <span class="_beta">CherryPick</span></template>
				<option value="reload">{{ i18n.ts._serverDisconnectedBehavior.reload }}</option>
				<option value="dialog">{{ i18n.ts._serverDisconnectedBehavior.dialog }}</option>
				<option value="quiet">{{ i18n.ts._serverDisconnectedBehavior.quiet }}</option>
				<option value="none">{{ i18n.ts._serverDisconnectedBehavior.none }}</option>
			</MkSelect>
			<MkSelect v-model="requireRefreshBehavior">
				<template #label>{{ i18n.ts.requireRefresh }} <span class="_beta">CherryPick</span></template>
				<option value="dialog">{{ i18n.ts._requireRefreshBehavior.dialog }}</option>
				<option value="quiet">{{ i18n.ts._requireRefreshBehavior.quiet }}</option>
			</MkSelect>
			<MkSelect v-model="newNoteReceivedNotificationBehavior">
				<template #label>{{ i18n.ts.newNoteReceivedNotification }} <span class="_beta">CherryPick</span></template>
				<option value="default">{{ i18n.ts._newNoteReceivedNotificationBehavior.default }}</option>
				<option value="count">{{ i18n.ts._newNoteReceivedNotificationBehavior.count }}</option>
				<option value="none">{{ i18n.ts._newNoteReceivedNotificationBehavior.none }}</option>
			</MkSelect>
			<MkRange v-model="numberOfPageCache" :min="1" :max="10" :step="1" easing>
				<template #label>{{ i18n.ts.numberOfPageCache }}</template>
				<template #caption>{{ i18n.ts.numberOfPageCacheDescription }}</template>
			</MkRange>

			<MkFolder>
				<template #label>{{ i18n.ts.dataSaver }}</template>

				<div class="_gaps_m">
					<MkInfo>{{ i18n.ts.reloadRequiredToApplySettings }}</MkInfo>

					<div class="_buttons">
						<MkButton inline @click="enableAllDataSaver">{{ i18n.ts.enableAll }}</MkButton>
						<MkButton inline @click="disableAllDataSaver">{{ i18n.ts.disableAll }}</MkButton>
					</div>
					<div class="_gaps_m">
						<MkSwitch v-model="dataSaver.media">
							{{ i18n.ts._dataSaver._media.title }}
							<template #caption>{{ i18n.ts._dataSaver._media.description }}</template>
						</MkSwitch>
						<MkSwitch v-model="dataSaver.avatar">
							{{ i18n.ts._dataSaver._avatar.title }}
							<template #caption>{{ i18n.ts._dataSaver._avatar.description }}</template>
						</MkSwitch>
						<MkSwitch v-model="dataSaver.urlPreview">
							{{ i18n.ts._dataSaver._urlPreview.title }}
							<template #caption>{{ i18n.ts._dataSaver._urlPreview.description }}</template>
						</MkSwitch>
						<MkSwitch v-model="dataSaver.code">
							{{ i18n.ts._dataSaver._code.title }}
							<template #caption>{{ i18n.ts._dataSaver._code.description }}</template>
						</MkSwitch>
					</div>
				</div>
			</MkFolder>
		</div>
	</FormSection>

	<FormSection>
		<template #label>{{ i18n.ts.other }}</template>

		<div class="_gaps">
			<MkFolder>
				<template #label>{{ i18n.ts.additionalEmojiDictionary }}</template>
				<div class="_buttons">
					<template v-for="lang in emojiIndexLangs" :key="lang">
						<MkButton v-if="defaultStore.reactiveState.additionalUnicodeEmojiIndexes.value[lang]" danger @click="removeEmojiIndex(lang)"><i class="ti ti-trash"></i> {{ i18n.ts.remove }} ({{ getEmojiIndexLangName(lang) }})</MkButton>
						<MkButton v-else @click="downloadEmojiIndex(lang)"><i class="ti ti-download"></i> {{ getEmojiIndexLangName(lang) }}{{ defaultStore.reactiveState.additionalUnicodeEmojiIndexes.value[lang] ? ` (${ i18n.ts.installed })` : '' }}</MkButton>
					</template>
				</div>
			</MkFolder>
			<FormLink to="/settings/deck">{{ i18n.ts.deck }}</FormLink>
			<FormLink to="/settings/custom-css"><template #icon><i class="ti ti-code"></i></template>{{ i18n.ts.customCss }}</FormLink>
		</div>
	</FormSection>
</div>
</template>

<script lang="ts" setup>
import { computed, onMounted, ref, watch } from 'vue';
import * as Misskey from 'cherrypick-js';
import MkSwitch from '@/components/MkSwitch.vue';
import MkSelect from '@/components/MkSelect.vue';
import MkRadios from '@/components/MkRadios.vue';
import MkRange from '@/components/MkRange.vue';
import MkFolder from '@/components/MkFolder.vue';
import MkButton from '@/components/MkButton.vue';
import FormSection from '@/components/form/section.vue';
import FormLink from '@/components/form/link.vue';
import MkLink from '@/components/MkLink.vue';
import MkInfo from '@/components/MkInfo.vue';
import { langs } from '@/config.js';
import { defaultStore } from '@/store.js';
import * as os from '@/os.js';
import { misskeyApi } from '@/scripts/misskey-api.js';
import { unisonReload } from '@/scripts/unison-reload.js';
import { i18n } from '@/i18n.js';
import { definePageMetadata } from '@/scripts/page-metadata.js';
import { miLocalStorage } from '@/local-storage.js';
import { globalEvents } from '@/events.js';
import { claimAchievement } from '@/scripts/achievements.js';

const lang = ref(miLocalStorage.getItem('lang'));
// const fontSize = ref(miLocalStorage.getItem('fontSize'));
const useSystemFont = ref(miLocalStorage.getItem('useSystemFont') != null);
const dataSaver = ref(defaultStore.state.dataSaver);

const fontSizeBefore = ref(miLocalStorage.getItem('fontSize'));
const useBoldFont = ref(miLocalStorage.getItem('useBoldFont'));

async function reloadAsk() {
	if (requireRefreshBehavior.value === 'dialog') {
		const { canceled } = await os.confirm({
			type: 'info',
			text: i18n.ts.reloadToApplySetting,
		});
		if (canceled) return;

		unisonReload();
	} else globalEvents.emit('hasRequireRefresh', true);
}

function reloadTimeline() {
	globalEvents.emit('reloadTimeline');
}

function reloadNotification() {
	globalEvents.emit('reloadNotification');
}

const hemisphere = computed(defaultStore.makeGetterSetter('hemisphere'));
const overridedDeviceKind = computed(defaultStore.makeGetterSetter('overridedDeviceKind'));
const serverDisconnectedBehavior = computed(defaultStore.makeGetterSetter('serverDisconnectedBehavior'));
const showNoteActionsOnlyHover = computed(defaultStore.makeGetterSetter('showNoteActionsOnlyHover'));
const showClipButtonInNoteFooter = computed(defaultStore.makeGetterSetter('showClipButtonInNoteFooter'));
const reactionsDisplaySize = computed(defaultStore.makeGetterSetter('reactionsDisplaySize'));
const limitWidthOfReaction = computed(defaultStore.makeGetterSetter('limitWidthOfReaction'));
const collapseRenotes = computed(defaultStore.makeGetterSetter('collapseRenotes'));
const reduceAnimation = computed(defaultStore.makeGetterSetter('animation', v => !v, v => !v));
const useBlurEffectForModal = computed(defaultStore.makeGetterSetter('useBlurEffectForModal'));
const useBlurEffect = computed(defaultStore.makeGetterSetter('useBlurEffect'));
const removeModalBgColorForBlur = computed(defaultStore.makeGetterSetter('removeModalBgColorForBlur'));
const showGapBetweenNotesInTimeline = computed(defaultStore.makeGetterSetter('showGapBetweenNotesInTimeline'));
const animatedMfm = computed(defaultStore.makeGetterSetter('animatedMfm'));
const advancedMfm = computed(defaultStore.makeGetterSetter('advancedMfm'));
const enableQuickAddMfmFunction = computed(defaultStore.makeGetterSetter('enableQuickAddMfmFunction'));
const emojiStyle = computed(defaultStore.makeGetterSetter('emojiStyle'));
const disableDrawer = computed(defaultStore.makeGetterSetter('disableDrawer'));
const disableShowingAnimatedImages = computed(defaultStore.makeGetterSetter('disableShowingAnimatedImages'));
const forceShowAds = computed(defaultStore.makeGetterSetter('forceShowAds'));
const loadRawImages = computed(defaultStore.makeGetterSetter('loadRawImages'));
const highlightSensitiveMedia = computed(defaultStore.makeGetterSetter('highlightSensitiveMedia'));
const imageNewTab = computed(defaultStore.makeGetterSetter('imageNewTab'));
const nsfw = computed(defaultStore.makeGetterSetter('nsfw'));
const showFixedPostForm = computed(defaultStore.makeGetterSetter('showFixedPostForm'));
const showFixedPostFormInChannel = computed(defaultStore.makeGetterSetter('showFixedPostFormInChannel'));
const numberOfPageCache = computed(defaultStore.makeGetterSetter('numberOfPageCache'));
const instanceTicker = computed(defaultStore.makeGetterSetter('instanceTicker'));
const enableInfiniteScroll = computed(defaultStore.makeGetterSetter('enableInfiniteScroll'));
const useReactionPickerForContextMenu = computed(defaultStore.makeGetterSetter('useReactionPickerForContextMenu'));
const squareAvatars = computed(defaultStore.makeGetterSetter('squareAvatars'));
const showAvatarDecorations = computed(defaultStore.makeGetterSetter('showAvatarDecorations'));
const mediaListWithOneImageAppearance = computed(defaultStore.makeGetterSetter('mediaListWithOneImageAppearance'));
const notificationPosition = computed(defaultStore.makeGetterSetter('notificationPosition'));
const notificationStackAxis = computed(defaultStore.makeGetterSetter('notificationStackAxis'));
const keepScreenOn = computed(defaultStore.makeGetterSetter('keepScreenOn'));
const disableStreamingTimeline = computed(defaultStore.makeGetterSetter('disableStreamingTimeline'));
const useGroupedNotifications = computed(defaultStore.makeGetterSetter('useGroupedNotifications'));
const enableSeasonalScreenEffect = computed(defaultStore.makeGetterSetter('enableSeasonalScreenEffect'));
const enableHorizontalSwipe = computed(defaultStore.makeGetterSetter('enableHorizontalSwipe'));
const showUnreadNotificationsCount = computed(defaultStore.makeGetterSetter('showUnreadNotificationsCount'));
const newNoteReceivedNotificationBehavior = computed(defaultStore.makeGetterSetter('newNoteReceivedNotificationBehavior'));
const fontSize = computed(defaultStore.makeGetterSetter('fontSize'));
const collapseDefault = computed(defaultStore.makeGetterSetter('collapseDefault'));
const requireRefreshBehavior = computed(defaultStore.makeGetterSetter('requireRefreshBehavior'));
const hideAvatarsInNote = computed(defaultStore.makeGetterSetter('hideAvatarsInNote'));
const showTranslateButtonInNote = computed(defaultStore.makeGetterSetter('showTranslateButtonInNote'));
const enableAbsoluteTime = computed(defaultStore.makeGetterSetter('enableAbsoluteTime'));
const enableMarkByDate = computed(defaultStore.makeGetterSetter('enableMarkByDate'));
const showSubNoteFooterButton = computed(defaultStore.makeGetterSetter('showSubNoteFooterButton'));
const infoButtonForNoteActionsEnabled = computed(defaultStore.makeGetterSetter('infoButtonForNoteActionsEnabled'));
const showReplyInNotification = computed(defaultStore.makeGetterSetter('showReplyInNotification'));
const renoteQuoteButtonSeparation = computed(defaultStore.makeGetterSetter('renoteQuoteButtonSeparation'));
const showFixedPostFormInReplies = computed(defaultStore.makeGetterSetter('showFixedPostFormInReplies'));
const showingAnimatedImages = computed(defaultStore.makeGetterSetter('showingAnimatedImages'));
const allMediaNoteCollapse = computed(defaultStore.makeGetterSetter('allMediaNoteCollapse'));
const nsfwOpenBehavior = computed(defaultStore.makeGetterSetter('nsfwOpenBehavior'));
const renoteVisibilitySelection = computed(defaultStore.makeGetterSetter('renoteVisibilitySelection'));
const forceRenoteVisibilitySelection = computed(defaultStore.makeGetterSetter('forceRenoteVisibilitySelection'));

watch(lang, () => {
	miLocalStorage.setItem('lang', lang.value as string);
	miLocalStorage.removeItem('locale');
	miLocalStorage.removeItem('localeVersion');
});

watch(fontSize, () => {
	if (fontSize.value == null) {
		miLocalStorage.removeItem('fontSize');
	} else {
		miLocalStorage.setItem('fontSize', fontSize.value as string);
	}
});

watch(useBoldFont, () => {
	if (useBoldFont.value) {
		miLocalStorage.setItem('useBoldFont', useBoldFont.value);
	} else {
		miLocalStorage.removeItem('useBoldFont');
	}
});

watch(useSystemFont, () => {
	if (useSystemFont.value) {
		miLocalStorage.setItem('useSystemFont', 't');
	} else {
		miLocalStorage.removeItem('useSystemFont');
	}
});

watch([
	hemisphere,
	lang,
	useBlurEffect,
	useBlurEffectForModal,
	removeModalBgColorForBlur,
	// fontSize,
	useBoldFont,
	useSystemFont,
	squareAvatars,
	showGapBetweenNotesInTimeline,
	overridedDeviceKind,
	keepScreenOn,
	disableStreamingTimeline,
	showUnreadNotificationsCount,
	showFixedPostFormInReplies,
	showingAnimatedImages,
	enableSeasonalScreenEffect,
], async () => {
	await reloadAsk();
});

watch([
	collapseDefault,
	hideAvatarsInNote,
	showNoteActionsOnlyHover,
	showClipButtonInNoteFooter,
	instanceTicker,
	mediaListWithOneImageAppearance,
	reactionsDisplaySize,
	limitWidthOfReaction,
	highlightSensitiveMedia,
	showReplyInNotification,
	showTranslateButtonInNote,
	enableAbsoluteTime,
	enableMarkByDate,
	showSubNoteFooterButton,
	infoButtonForNoteActionsEnabled,
	renoteQuoteButtonSeparation,
	allMediaNoteCollapse,
], () => {
	reloadTimeline();
	reloadNotification();
});

watch([
	collapseRenotes,
	enableInfiniteScroll,
], () => {
	reloadTimeline();
});

watch([
	showReplyInNotification,
], () => {
	reloadNotification();
});

const emojiIndexLangs = ['en-US', 'ja-JP', 'ja-JP_hira'] as const;

function getEmojiIndexLangName(targetLang: typeof emojiIndexLangs[number]) {
	if (langs.find(x => x[0] === targetLang)) {
		return langs.find(x => x[0] === targetLang)![1];
	} else {
		// 絵文字辞書限定の言語定義
		switch (targetLang) {
			case 'ja-JP_hira': return 'ひらがな';
			default: return targetLang;
		}
	}
}

function downloadEmojiIndex(lang: typeof emojiIndexLangs[number]) {
	async function main() {
		const currentIndexes = defaultStore.state.additionalUnicodeEmojiIndexes;

		function download() {
			switch (lang) {
				case 'en-US': return import('../../unicode-emoji-indexes/en-US.json').then(x => x.default);
				case 'ja-JP': return import('../../unicode-emoji-indexes/ja-JP.json').then(x => x.default);
				case 'ja-JP_hira': return import('../../unicode-emoji-indexes/ja-JP_hira.json').then(x => x.default);
				default: throw new Error('unrecognized lang: ' + lang);
			}
		}

		currentIndexes[lang] = await download();
		await defaultStore.set('additionalUnicodeEmojiIndexes', currentIndexes);
	}

	os.promiseDialog(main());
}

function removeEmojiIndex(lang: string) {
	async function main() {
		const currentIndexes = defaultStore.state.additionalUnicodeEmojiIndexes;
		delete currentIndexes[lang];
		await defaultStore.set('additionalUnicodeEmojiIndexes', currentIndexes);
	}

	os.promiseDialog(main());
}

async function setPinnedList() {
	const lists = await misskeyApi('users/lists/list');
	const { canceled, result: list } = await os.select({
		title: i18n.ts.selectList,
		items: lists.map(x => ({
			value: x, text: x.name,
		})),
	});
	if (canceled) return;

	defaultStore.set('pinnedUserLists', [list]);
}

function removePinnedList() {
	defaultStore.set('pinnedUserLists', []);
}

let smashCount = 0;
let smashTimer: number | null = null;

function testNotification(): void {
	const notification: Misskey.entities.Notification = {
		id: Math.random().toString(),
		createdAt: new Date().toUTCString(),
		isRead: false,
		type: 'test',
	};

	globalEvents.emit('clientNotification', notification);

	// セルフ通知破壊 実績関連
	smashCount++;
	if (smashCount >= 10) {
		claimAchievement('smashTestNotificationButton');
		smashCount = 0;
	}
	if (smashTimer) {
		clearTimeout(smashTimer);
	}
	smashTimer = window.setTimeout(() => {
		smashCount = 0;
	}, 300);
}

function enableAllDataSaver() {
	const g = { ...defaultStore.state.dataSaver };
	Object.keys(g).forEach((key) => { g[key] = true; });
	dataSaver.value = g;
}

function disableAllDataSaver() {
	const g = { ...defaultStore.state.dataSaver };
	Object.keys(g).forEach((key) => { g[key] = false; });
	dataSaver.value = g;
}

watch(dataSaver, (to) => {
	defaultStore.set('dataSaver', to);
}, {
	deep: true,
});

onMounted(() => {
	if (fontSizeBefore.value == null) {
		fontSizeBefore.value = fontSize.value as string;
	}
});

const headerActions = computed(() => []);

const headerTabs = computed(() => []);

definePageMetadata(() => ({
	title: i18n.ts.general,
	icon: 'ti ti-adjustments',
}));
</script>

<style lang="scss" module>
.label {
	font-size: 0.85em;
	padding: 0 0 8px 0;
	user-select: none;
}

.mfmPreview, .fontSize {
	padding: 20px 20px 28px;
	border-radius: 6px;
	text-align: center;
}

.mfmPreview {
	padding: 5px;
	max-width: 110px;
}

.fontSizeSlider {
	display: flex;
	margin-top: -8px;
	border-top: solid .5px var(--divider);

	> .fontSizeLeft, .fontSizeRight {
		position: relative;
		background: var(--panel);
		font-weight: normal;
		line-height: 20px;
	}

	> .fontSizeLeft {
		padding: 7px 6px 7px 18px;
		border-bottom-left-radius: 6px;
		font-size: 12px;
	}

	> .fontSizeRight {
		padding: 7px 18px 7px 6px;
		border-bottom-right-radius: 6px;
		font-size: 18px;
	}
}
</style>
