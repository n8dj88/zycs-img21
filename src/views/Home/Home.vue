<template>
  <section class="Home pt-4 sm:pt-6">
    <!-- 顶部提示公告 -->
    <Alert class="pt-0 pb-2 sm:py-4">
      <AlertTitle class="font-bold hidden sm:flex sm:gap-2">
        <RocketIcon class="h-4 w-4 hidden sm:flex" />
        Heads up!
      </AlertTitle>
      <AlertDescription class="p-0 text-xs sm:text-sm">
        <p class="pt-2">无限图片储存数量，你可以上传不限数量的图片！</p>
        <p>图片首次访问后缓存，"永久"有效，包括全球分布的 CDN，以确保尽可能快地提供图像.</p>
        <p>支持并维护的文件上传项目，致力于为用户提供稳定的永久存储服务。</p>
      </AlertDescription>
    </Alert>

    <!-- 顶部广告空位 -->
    <div class="ad-top my-4 rounded-lg border border-dashed border-slate-300 p-3 bg-slate-50 text-center">
      <a href="" target="_blank" rel="noopener noreferrer">
        <img src="" alt="广告位" class="max-w-full h-auto" />
      </a>
    </div>

    <!-- 图床切换工具栏 -->
    <div class="pt-6 flex items-center text-sm">
      <div class="sync shrink-0">
        <RadioGroup default-value="sync" class="flex items-center gap-4 [&>label]:flex [&>label]:items-center [&>label]:space-x-2 [&>label]:cursor-pointer">
          <Label for="sync">
            <RadioGroupItem id="sync" value="sync" />
            <span>Imgur</span>
          </Label>
          <Label for="nosync">
            <RadioGroupItem id="nosync" value="nosync" disabled />
            <span class="text-gray-300">待定</span>
          </Label>
        </RadioGroup>
      </div>
    </div>

    <!-- 上传组件区域 -->
    <Upload v-model="fileList" :UploadConfig="UploadConfig" :uploadAPI="uploadAPI" />

    <!-- 批量操作工具栏（仅上传成功后显示） -->
    <section v-show="fileList.length" class="vh-tools mt-3 flex gap-2">
      <Button @click="handleClearAll">清空全部</Button>
      <Button @click="handleCopyAll">复制全部链接</Button>
    </section>

    <!-- 图片结果列表 -->
    <ResList v-model="fileList" :nodeHost="nodeHost" />

    <!-- 底部广告空位 -->
    <div class="ad-bottom mt-6 rounded-lg border border-dashed border-slate-300 p-3 bg-slate-50 text-center">
      <a href="" target="_blank" rel="noopener noreferrer">
        <img src="" alt="底部广告位" class="max-w-full h-auto" />
      </a>
    </div>
  </section>
</template>

<script setup lang="ts">
// 第三方依赖
import { ref, watch, type WatchCallback } from 'vue'
import { RocketIcon } from '@radix-icons/vue'
import vh from 'vh-plugin'

// 全局工具
import { formatURL } from '@/utils/index'

// UI组件库
import { Button } from '@/components/ui/button'
import { Label } from '@/components/ui/label'
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'
import { RadioGroup, RadioGroupItem } from '@/components/ui/radio-group'

// 业务组件
import Upload from '@/components/Upload/Upload.vue'
import ResList from '@/components/ResList/ResList.vue'

// ==================== 类型定义 ====================
export interface FileItem {
  upload_status: 'success' | 'loading' | 'error'
  upload_result: string
  upload_blob?: string
}

// ==================== 常量抽离 ====================
const STORAGE_KEY = 'zychUpImageList'
const API_BASE = import.meta.env.VITE_IMG_API_URL || location.origin

// ==================== 响应式数据 ====================
const nodeHost = ref<string>(API_BASE)
const uploadAPI = ref<string>(`${API_BASE}/upload`)

// 上传配置
const UploadConfig = ref({
  AcceptTypes: 'image/*',
  Max: 0,
  MaxSize: 15
})

// 读取本地缓存图片列表，增加JSON解析容错
const getCacheList = (): FileItem[] => {
  try {
    const cache = localStorage.getItem(STORAGE_KEY)
    return cache ? JSON.parse(cache) : []
  } catch {
    return []
  }
}
const fileList = ref<FileItem[]>(getCacheList())

// ==================== 业务方法 ====================
/** 清空上传列表 */
const handleClearAll = () => {
  fileList.value = []
}

/** 复制全部成功图片链接，空列表容错 */
const handleCopyAll = () => {
  const linkList = fileList.value
    .filter(item => item.upload_status === 'success')
    .map(item => item.upload_blob)
    .filter(Boolean) as string[]

  if (!linkList.length) return
  vh.CopyText(linkList.join('\n'))
}

/** 监听列表变化持久化本地存储 */
const saveListToStorage: WatchCallback<FileItem[]> = (newVal) => {
  const storeData = newVal
    .filter(item => item.upload_status === 'success')
    .map(item => {
      item.upload_blob = formatURL({ nodeHost: nodeHost.value }, item.upload_result)
      return item
    })
  localStorage.setItem(STORAGE_KEY, JSON.stringify(storeData))
}

watch(fileList, saveListToStorage, { deep: true })
</script>

<style scoped lang="less">
@import 'Home.less';
.ad-top {
  min-height: 70px;
}
.ad-bottom {
  min-height: 70px;
}
</style>
