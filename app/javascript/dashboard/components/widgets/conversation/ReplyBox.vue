<template>
  <Banner
    v-if="showSelfAssignBanner"
    action-button-variant="clear"
    color-scheme="secondary"
    class="banner--self-assign mx-2 mb-2 rounded-lg"
    :banner-message="$t('CONVERSATION.NOT_ASSIGNED_TO_YOU')"
    has-action-button
    :action-button-label="$t('CONVERSATION.ASSIGN_TO_ME')"
    @primary-action="onClickSelfAssign"
  />
  <div ref="replyEditor" class="reply-box" :class="replyBoxClass">
    <ReplyTopPanel
      :mode="replyType"
      :is-message-length-reaching-threshold="isMessageLengthReachingThreshold"
      :characters-remaining="charactersRemaining"
      :popout-reply-box="popoutReplyBox"
      @set-reply-mode="setReplyMode"
      @toggle-popout="$emit('togglePopout')"
    />
    <ArticleSearchPopover
      v-if="showArticleSearchPopover && connectedPortalSlug"
      :selected-portal-slug="connectedPortalSlug"
      @insert="handleInsert"
      @close="onSearchPopoverClose"
    />
    <div class="reply-box__top">
      <ReplyToMessage
        v-if="shouldShowReplyToMessage"
        :message="inReplyTo"
        @dismiss="resetReplyToMessage"
      />
      <CannedResponse
        v-if="showMentions && hasSlashCommand"
        v-on-clickaway="hideMentions"
        class="normal-editor__canned-box"
        :search-key="mentionSearchKey"
        @replace="replaceText"
      />
      <EmojiInput
        v-if="showEmojiPicker"
        v-on-clickaway="hideEmojiPicker"
        :class="emojiDialogClassOnExpandedLayoutAndRTLView"
        :on-click="addIntoEditor"
      />
      <ReplyEmailHead
        v-if="showReplyHead"
        v-model:cc-emails="ccEmails"
        v-model:bcc-emails="bccEmails"
        v-model:to-emails="toEmails"
      />
      <AudioRecorder
        v-if="showAudioRecorderEditor"
        ref="audioRecorderInput"
        :audio-record-format="audioRecordFormat"
        @recorder-progress-changed="onRecordProgressChanged"
        @finish-record="onFinishRecorder"
        @play="recordingAudioState = 'playing'"
        @pause="recordingAudioState = 'paused'"
      />
      <ResizableTextArea
        v-else-if="!showRichContentEditor"
        ref="messageInput"
        v-model="message"
        class="input"
        :placeholder="messagePlaceHolder"
        :min-height="4"
        :signature="signatureToApply"
        allow-signature
        :send-with-signature="sendWithSignature"
        @typing-off="onTypingOff"
        @typing-on="onTypingOn"
        @focus="onFocus"
        @blur="onBlur"
      />
      <WootMessageEditor
        v-else
        v-model="message"
        :editor-id="editorStateId"
        class="input"
        :is-private="isOnPrivateNote"
        :placeholder="messagePlaceHolder"
        :update-selection-with="updateEditorSelectionWith"
        :min-height="4"
        enable-variables
        :variables="messageVariables"
        :signature="signatureToApply"
        allow-signature
        :channel-type="channelType"
        @typing-off="onTypingOff"
        @typing-on="onTypingOn"
        @focus="onFocus"
        @blur="onBlur"
        @toggle-user-mention="toggleUserMention"
        @toggle-canned-menu="toggleCannedMenu"
        @toggle-variables-menu="toggleVariablesMenu"
        @clear-selection="clearEditorSelection"
      />
    </div>
    <div
      v-if="hasAttachments && !showAudioRecorderEditor"
      class="attachment-preview-box"
      @paste="onPaste"
    >
      <AttachmentPreview
        class="flex-col mt-4"
        :attachments="attachedFiles"
        @remove-attachment="removeAttachment"
      />
    </div>
    <MessageSignatureMissingAlert
      v-if="isSignatureEnabledForInbox && !isSignatureAvailable"
    />
    <ReplyBottomPanel
      :conversation-id="conversationId"
      :enable-multiple-file-upload="enableMultipleFileUpload"
      :has-whatsapp-templates="hasWhatsappTemplates"
      :inbox="inbox"
      :is-on-private-note="isOnPrivateNote"
      :is-recording-audio="isRecordingAudio"
      :is-send-disabled="isReplyButtonDisabled"
      :mode="replyType"
      :on-file-upload="onFileUpload"
      :on-send="onSendReply"
      :recording-audio-duration-text="recordingAudioDurationText"
      :recording-audio-state="recordingAudioState"
      :send-button-text="replyButtonLabel"
      :show-audio-recorder="showAudioRecorder"
      :show-editor-toggle="isAPIInbox && !isOnPrivateNote"
      :show-emoji-picker="showEmojiPicker"
      :show-file-upload="showFileUpload"
      :toggle-audio-recorder-play-pause="toggleAudioRecorderPlayPause"
      :toggle-audio-recorder="toggleAudioRecorder"
      :toggle-emoji-picker="toggleEmojiPicker"
      :message="message"
      :portal-slug="connectedPortalSlug"
      :new-conversation-modal-active="newConversationModalActive"
      @select-whatsapp-template="openWhatsappTemplateModal"
      @toggle-editor="toggleRichContentEditor"
      @replace-text="replaceText"
      @toggle-insert-article="toggleInsertArticle"
    />
    <WhatsappTemplates
      :inbox-id="inbox.id"
      :show="showWhatsAppTemplatesModal"
      @close="hideWhatsappTemplatesModal"
      @on-send="onSendWhatsAppReply"
      @cancel="hideWhatsappTemplatesModal"
    />

    <woot-confirm-modal
      ref="confirmDialog"
      :title="$t('CONVERSATION.REPLYBOX.UNDEFINED_VARIABLES.TITLE')"
      :description="undefinedVariableMessage"
    />
  </div>
</template>

<style lang="scss" scoped>
.send-button {
  @apply mb-0;
}

.banner--self-assign {
  @apply py-2;
}

.attachment-preview-box {
  @apply bg-transparent py-2 px-4 border-t border-slate-800/30;
}

.reply-box {
  transition: height 0.2s ease-in-out;
  @apply relative mb-2 mx-2 border border-slate-800/30 rounded-xl bg-slate-800/50;

  &.is-private {
    @apply bg-amber-500/10 border-amber-500/20;
  }

  &.is-focused {
    @apply border-violet-500/30 bg-slate-800;
  }
}

.reply-box__top {
  @apply relative py-2 px-4;

  textarea {
    @apply shadow-none border-transparent bg-transparent m-0 max-h-60 min-h-[3rem] pt-2 pb-0 px-0 resize-none text-slate-100 placeholder-slate-400;

    &:focus {
      @apply outline-none ring-0 border-transparent;
    }
  }
}

.emoji-dialog {
  @apply top-[unset] -bottom-10 -left-80 right-[unset] bg-slate-800 border border-slate-700/50 shadow-lg;

  &::before {
    transform: rotate(270deg);
    filter: drop-shadow(0px 4px 4px rgba(0, 0, 0, 0.2));
    @apply -right-4 bottom-2 rtl:right-0 rtl:-left-4;
  }
}

.emoji-dialog--rtl {
  @apply left-[unset] -right-80;

  &::before {
    transform: rotate(90deg);
    filter: drop-shadow(0px 4px 4px rgba(0, 0, 0, 0.2));
  }
}

.emoji-dialog--expanded {
  @apply left-[unset] bottom-0 absolute z-[100];

  &::before {
    transform: rotate(0deg);
    @apply left-1 -bottom-2;
  }
}

.normal-editor__canned-box {
  width: calc(100% - 2 * var(--space-normal));
  left: var(--space-normal);
}
</style>
