<template>
  <div id="subApp"></div>
</template>

<script setup lang="ts">
import MountLoginInstance from './instances/login/login';
import MountAdminInstance from '@admin/admin.ts';
import { unref, watchPostEffect } from 'vue';
import { useModifyInstanceType } from '@/hooks';
import type { InstanceType } from '@/hooks';
import type { App } from 'vue';
import { switchTheme } from '@/theme';

const { instanceType } = useModifyInstanceType();

/**
 * 实例对象，键为实例类型，值为对应实例的函数
 */
const instanceMap: Map<InstanceType, InstanceRenderFun> = new Map([
  ['login', MountLoginInstance],
  ['admin', MountAdminInstance],
]);

let instance: App;

async function render() {
  const type = unref(instanceType) || 'login'; // 添加一个login默认值 防止手动修改了本地存放的实例类型导致找不到实例函数
  // 根据实例类型获取对应的实例函数
  const instanceFunction = instanceMap.get(type);

  if (instanceFunction) {
    instance && instance.unmount(); // 挂载新的实例之前 卸载旧实例
    await switchTheme('purple'); // 默认紫色主题 //todo 可能需要优化 后面在想想主题的设置
    instance = await instanceFunction();
  }
}

/**
 * 渲染中间动画
 * 流程: 动画 渲染 动画
 */
async function applyAnimation() {
  await render();
}

watchPostEffect(async () => {
  await applyAnimation();
});
</script>
