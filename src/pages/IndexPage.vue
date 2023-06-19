<template>
  <div class="index-page">
    <a-input-search
      v-model:value="searchText"
      placeholder="input search text"
      enter-button="Search"
      size="large"
      @search="onSearch"
    />
    <MyDivider />
    <a-tabs v-model:activeKey="activeKey" @change="onTabChange">
      <a-tab-pane key="post" tab="文章">
        <PostList :post-list="postList" />
      </a-tab-pane>
      <a-tab-pane key="picture" tab="图片">
        <PictureList :picture-list="pictureList" />
      </a-tab-pane>
      <a-tab-pane key="user" tab="用户">
        <UserList :user-list="userList" />
      </a-tab-pane>
    </a-tabs>
  </div>
</template>

<script setup lang="ts">
import { ref, watchEffect } from "vue";
import PostList from "@/components/PostList.vue";
import PictureList from "@/components/PictureList.vue";
import UserList from "@/components/UserList.vue";
import MyDivider from "@/components/MyDivider.vue";
import { useRoute, useRouter } from "vue-router";
import myAxios from "@/plugins/myAxios";
import { message } from "ant-design-vue";

const postList = ref([]);

const pictureList = ref([]);

const userList = ref([]);

const router = useRouter();
const route = useRoute();
const activeKey = route.params.category;

// 搜索栏初始值
const initSearchParams = {
  type: activeKey,
  text: "",
  pageSize: 10,
  pageNum: 1,
};

const searchText = ref(route.query.text || "");

/**
 * 加载数据
 * @param params
 */
const loadDataOld = (params: any) => {
  const postQuery = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("/post/list/page/vo", { postQuery }).then((res: any) => {
    postList.value = res.records;
  });

  const pictureQuery = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("/picture/list/page/vo", { pictureQuery }).then((res: any) => {
    pictureList.value = res.records;
  });

  const userQuery = {
    ...params,
    userName: params.text,
  };
  myAxios.post("/user/list/page/vo", { userQuery }).then((res: any) => {
    userList.value = res.records;
  });
};

/**
 * 加载聚合数据
 * @param params
 */
const loadAllData = (params: any) => {
  const query = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("/search/all", { query }).then((res: any) => {
    postList.value = res.postList;
    pictureList.value = res.pictureList;
    userList.value = res.userList;
  });
};

/**
 * 加载单类数据
 * @param params
 */
const loadData = (params: any) => {
  const { type } = params;
  if (!type) {
    message.error("类别为空");
    return;
  }
  const query = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("/search/all", { ...query }).then((res: any) => {
    // alert(type);
    if (type === "post") {
      postList.value = res.dataList;
    } else if (type === "user") {
      userList.value = res.dataList;
    } else if (type === "picture") {
      pictureList.value = res.dataList;
    }
  });
};

// 首次请求
// loadData(initSearchParams);

// 搜索栏上的默认值
const searchParams = ref(initSearchParams);

// 监听：里边的任何参数修改都会重新执行
// watchEffect自动收集依赖数据，依赖数据更新时重新执行自身
watchEffect(() => {
  searchParams.value = {
    ...initSearchParams,
    text: route.query.text,
    type: route.params.category,
  } as any;
  loadData(searchParams.value);
});

// 点击搜索：利用query把搜索栏的值也添加到url上去
const onSearch = (value: string) => {
  console.log(value);
  router.push({
    query: {
      ...searchParams.value,
      text: value,
    },
  });
};

// 面板改变事件：当面板改变的时候将面板的值强制添加到url地址上去，利用query把搜索栏的值也添加到url上去
const onTabChange = (key: string) => {
  router.push({
    path: `/${key}`,
    query: searchParams.value,
  });
};
</script>
