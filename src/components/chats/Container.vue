<template>
  <div
    class="h-full bg-white md:max-w-sm w-full relative"
    :style="convertToCss(story.style?.general)"
  >
    <div
      class="px-5 absolute top-0 w-full h-14 flex items-center"
      :style="convertToCss(story.style.navigation)"
    >
      <p class="flex-1 text-overflow">{{ story.title }}</p>
      <slot name="header"></slot>
    </div>
    <div ref="chatContainer" class="pb-5 px-2 pt-16 overflow-auto scroll h-full">
      <component
        :is="chat.type"
        v-for="(chat, index) in state.chats"
        :key="index"
        v-bind="{ data: chat, style: story.style }"
      ></component>
      <div v-if="state.options.length !== 0" class="pl-10 text-right space-y-2 mt-4">
        <div v-for="option in state.options" :key="option.id" class="option">
          <p
            class="p-2 relative inline-block animate-pulse cursor-pointer"
            :style="convertToCss(story.style.option)"
            :title="option.text"
            style="min-width: 4rem"
            @click="optionHandler(option)"
          >
            {{ option.text }}
          </p>
        </div>
      </div>
    </div>
    <div class="transition fade"></div>
  </div>
</template>
<script>
import { onMounted, reactive, computed, onUnmounted, onUpdated, ref } from 'vue';
import { convertToCss } from '@/utils/helper';
import Chat from './Chat.vue';
import Annotation from './Annotation.vue';
import Ending from './Ending.vue';
import StoryEngine from '@/utils/storyEngine';
import Story from '@/models/story';

export default {
  components: { Chat, Annotation, Ending },
  props: {
    storyId: {
      type: String,
      default: '',
      require: true,
    },
    progress: {
      type: Array,
      default: () => [],
    },
  },
  emits: ['end'],
  setup(props, { emit }) {
    const state = reactive({
      engine: null,
      chats: [],
      options: [],
      selectedOptions: [],
    });
    const chatContainer = ref(null);

    const story = computed(() => Story.query().where('id', props.storyId).withAll().first());

    function optionHandler({ id, text }) {
      state.selectedOptions.push(id);
      state.options = [];
      state.engine.addChat({
        type: 'chat',
        isMainCharacter: true,
        message: text,
        characterId: story.value.mainCharacter,
        imageUrl: '',
      });
      state.engine.start(id);
    }

    onUpdated(() => {
      chatContainer.value.scrollTop = chatContainer.value.scrollHeight;
    });
    onMounted(() => {
      const storyEngine = new StoryEngine(story.value, chatContainer.value);

      state.engine = storyEngine;

      if (story.value.progress && story.value.progress.options.length !== 0) {
        storyEngine.load(story.value.progress.options);
      } else {
        const start = story.value.nodes.find((node) => node.type === 'start');
        storyEngine.start(start.id);
      }

      storyEngine.on('options', (options) => {
        state.options = options;
      });
      storyEngine.on('chat-added', (chat) => {
        state.chats.push(chat);
      });
      storyEngine.on('end', () => {
        emit('end', true);
      });
      storyEngine.on('progress-loaded', (chats) => {
        state.chats = chats;
      });

      state.engine = storyEngine;
    });
    onUnmounted(() => {
      state.engine.destroy();
    });

    return {
      state,
      story,
      convertToCss,
      optionHandler,
      chatContainer,
    };
  },
};
</script>
