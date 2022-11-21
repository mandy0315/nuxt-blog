<template>
  <main class="c-container pt-6 pb-10">
    <div class="flex items-center justify-between">
      <h1 class="c-dashboard-title">文章管理</h1>
      <nuxt-link to="/dashboard/post-edit" class="c-rounded-button c-rounded-button-gray">+ 新增文章</nuxt-link>
    </div>
    <ul class="mt-6">
      <li
        v-for="item in postsStateList"
        :key="item.state"
        class="mr-1 inline-block rounded-t-lg bg-white py-1 px-4 text-lg text-c-gray-800 hover:opacity-100"
        :class="currState === item.state ? 'opacity-100' : 'opacity-50'"
      >
        <nuxt-link :to="`/dashboard/posts-${item.state}`">{{ item.name }}</nuxt-link>
      </li>
    </ul>

    <section class="w-full rounded-b-md rounded-tr-md bg-white p-6">
      <table class="w-full">
        <thead class="bg-c-gray-800 text-white">
          <tr>
            <th scope="row" class="w-5/6 py-2">文章資訊</th>
            <th scope="row" class="w-1/6 py-2">管理</th>
          </tr>
        </thead>
        <tbody v-if="postsList.length > 0">
          <tr v-for="item in postsList" :key="item.id" class="border-b border-solid border-c-gray-400">
            <td scope="col" class="p-4 text-c-gray-600">
              <post-list v-bind="item" />
            </td>
            <td class="p-4">
              <nuxt-link
                :to="`/dashboard/post-edit/${item.id}`"
                class="c-border-button c-border-button-gray my-1 w-full text-center"
              >
                編輯
              </nuxt-link>
              <button
                @click.prevent="openPreviewPost(item.id)"
                class="c-border-button c-border-button-gray my-1 w-full"
              >
                預覽
              </button>
              <button @click.prevent="deleteCase(item.id)" class="c-border-button c-border-button-red my-1 w-full">
                刪除
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </section>
  </main>
</template>

<script setup>
import { $vfm } from 'vue-final-modal';

import CustomModal from '@/components/customModal.vue';
import PostContent from '@/components/post/content.vue';

useHead({ title: '文章管理' });
definePageMeta({
  middleware: [
    (to, form) => {
      if (!['public', 'draft'].includes(to.params.state)) {
        return navigateTo('/dashboard/posts-public');
      }
    }
  ]
});
const postsStateList = [
  {
    state: 'public',
    name: '公開'
  },
  {
    state: 'draft',
    name: '草稿'
  }
];

const route = useRoute();
const currState = computed(() => route.params.state);
const postsList = useState(() => []);
const { getPostsPublicListAPI, getPostsDraftListAPI, deletePostsAPI, getPostsAPI } = useFirebase();

// 初始-取的資料
const getPostsListData = {
  public: async () => {
    const data = await getPostsPublicListAPI();
    return data.result;
  },
  draft: async () => {
    const data = await getPostsDraftListAPI();
    return data.result;
  }
};
const getPostsList = async () => {
  const data = await getPostsListData[currState.value]();
  postsList.value = data || [];
};
getPostsList();

const deleteCase = async id => {
  const data = await deletePostsAPI(id);
  data.success && getPostsList();
};

const openPreviewPost = async id => {
  const data = await getPostsAPI(id);
  data.success &&
    $vfm.show({
      component: CustomModal,
      bind: {
        modalContainerClass: 'max-w-[960px]',
        modalContentClass: 'p-6'
      },
      on: {},
      slots: {
        default: {
          component: PostContent,
          bind: {
            title: data.result.title,
            categories: data.result.categories,
            content: data.result.content,
            update_time: data.result.update_time
          }
        }
      }
    });
};
</script>