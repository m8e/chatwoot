<template>
  <UserMessage v-if="isUserMessage" :message="message.content" />
  <AgentMessage
    v-else
    :agent-name="message.sender_name"
    :message="message.content"
  />
</template>

<script>
import AgentMessage from 'widget/components/AgentMessage.vue';
import UserMessage from 'widget/components/UserMessage.vue';
import { MESSAGE_TYPE } from 'widget/helpers/constants';

export default {
  components: {
    AgentMessage,
    UserMessage,
  },
  props: {
    message: Object,
  },
  computed: {
    isUserMessage() {
      return this.message.message_type === MESSAGE_TYPE.INCOMING;
    },
  },
};
</script>

<style scoped lang="scss">
.message-wrap {
  display: flex;
  flex-direction: row;
  align-items: flex-end;
  max-width: 90%;
}
</style>
