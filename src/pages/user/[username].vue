<template>
  <div class="container py-12">
    <div v-if="state.loading" class="text-center">
      <ui-spinner size="36"></ui-spinner>
    </div>
    <template v-else>
      <div class="flex items-center">
        <img
          :src="state.user.photoURL ?? `https://ui-avatars.com/api/?name=${state.user.username}`"
          class="h-14 w-14 rounded-full"
        />
        <p class="ml-4 text-2xl font-semibold">{{ $route.params.username }}</p>
      </div>
      <p class="mt-12 mb-5 font-semibold text-xl">Stories</p>
      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
        <ui-story-card
          v-for="story in state.stories"
          :key="story.id"
          v-bind="{ story }"
        ></ui-story-card>
      </div>
      <div v-if="state.nextKey" class="text-center mt-12">
        <ui-button :loading="state.loadingLoadMore" @click="loadMore">Load more stories</ui-button>
      </div>
    </template>
  </div>
</template>
<script>
import { computed, onMounted, shallowReactive } from 'vue';
import { useRouter } from 'vue-router';
import { useHead } from '@vueuse/head';
import { fetchAPI } from '@/utils/auth';
import Story from '@/models/story';

export default {
  setup() {
    const router = useRouter();
    const state = shallowReactive({
      user: {},
      nextKey: null,
      loading: true,
      stories: [],
      loadingLoadMore: false,
    });

    const { username } = router.currentRoute.value.params;

    useHead({
      title: `${username} | Chat Story`,
    });

    async function loadMore() {
      try {
        state.loadingLoadMore = true;
        const { stories, nextKey } = await fetchAPI(
          `/story?sortBy=recently&author=${state.user._id}&nextPageId=${JSON.stringify(
            state.nextKey
          )}`
        );

        state.stories.push(...stories);
        state.nextKey = nextKey;
        state.loadingLoadMore = false;
      } catch (error) {
        state.loadingLoadMore = false;
        console.error(error);
      }
    }

    onMounted(async () => {
      state.loading = true;

      try {
        const user = await fetchAPI(`/user/${username}`);
        const { stories, nextKey } = await fetchAPI(`/story?sortBy=recently&author=${user._id}`);

        state.user = user;
        state.stories = stories;
        state.nextKey = nextKey;
        state.loading = false;
      } catch (error) {
        router.replace('/404');
        console.error(error);
      }
    });

    return {
      state,
      loadMore,
    };
  },
};
</script>
