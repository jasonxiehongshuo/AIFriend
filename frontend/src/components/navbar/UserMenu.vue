<script setup>
import { useUserStore } from "@/stores/user.js";
import UserSpaceIndex from "@/components/navbar/icons/UserSpaceIndex.vue";
import UserProfileIcon from "@/components/navbar/icons/UserProfileIcon.vue";
import UserLogoutIcon from "@/components/navbar/icons/UserLogoutIcon.vue";
import api from "@/js/http/api.js";
import { useRouter } from "vue-router";
// 新增：导入组件销毁钩子，用于处理未完成的请求
import { onUnmounted } from 'vue';

const user = useUserStore()
const router = useRouter()

// 新增：创建请求取消控制器，防止组件销毁导致请求端口异常
const abortController = new AbortController();

function closeMenu() {
  const element = document.activeElement
  if (element && element instanceof HTMLElement) element.blur()
}

async function handleLogout() {
  try {
    // 修改：给请求添加取消信号，同时补全请求参数格式（适配多数 axios 封装）
    const res = await api.post(
      '/api/user/account/logout/',
      {}, // 请求体（无参数时传空对象）
      { signal: abortController.signal } // 绑定取消信号
    );
    if (res.data.result === 'success') {
      user.logout();
      // 修改：延迟跳转，给请求端口留收尾时间（避免端口提前关闭）
      setTimeout(() => {
        router.push({ name: 'homepage-index' });
      }, 100);
    }
  } catch (err) {
    // 修改：过滤「请求被取消」的无害异常，只打印真实错误
    if (err.name !== 'AbortError') {
      console.error('退出登录失败：', err);
    }
  }
}

// 新增：组件销毁时主动取消未完成的请求，避免端口异常
onUnmounted(() => {
  abortController.abort();
});
</script>

<template>
  <div class="dropdown dropdown-end">
    <div tabindex="0" role="button" class="avatar btn btn-circle w-8 h-8 mr-6">
      <div class="w-8 rounded-full">
        <img :src="user.photo" alt="用户头像">
      </div>
    </div>
    <ul tabindex="-1" class="dropdown-content menu bg-base-100 rounded-box z-1 w-52 p-2 shadow-sm">
      <li>
        <RouterLink @click="closeMenu" :to="{name: 'user-space-index',params:{user_id: user.id}}">
          <div class="avatar">
            <div class="w-10 rounded-full">
              <img :src="user.photo" alt="用户头像">
            </div>
          </div>
          <span class="text-base font-bold line-clamp-1">{{user.username}}</span>
        </RouterLink>
      </li>
      <li>
        <!-- 修复：typo 错误 front-bold → font-bold，保证样式生效 -->
        <RouterLink @click="closeMenu" :to="{name: 'user-space-index',params:{user_id: user.id}}" class="text-sm font-bold py-3">
          <UserSpaceIndex />
          个人空间
        </RouterLink>
      </li>
      <li>
        <!-- 修复：typo 错误 front-bold → font-bold -->
        <RouterLink @click="closeMenu" :to="{name: 'user-profile-index'}" class="text-sm font-bold py-3">
          <UserProfileIcon />
          编辑资料
        </RouterLink>
      </li>
      <li/>
      <li>
        <!-- 修复：typo 错误 front-bold → font-bold -->
        <a @click="handleLogout" class="text-sm font-bold py-3">
          <UserLogoutIcon />
          退出登录
        </a>
      </li>
    </ul>
  </div>
</template>
<style scoped>
</style>