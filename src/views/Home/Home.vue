<template>
  <section class="Home pt-4 sm:pt-6">
    <Alert class="pt-0 pb-2 sm:py-4">
      <AlertTitle class="font-bold hidden sm:flex sm:gap-2">
        <RocketIcon class="h-4 w-4 hidden sm:flex" /> Heads up!
      </AlertTitle>
      <AlertDescription class="p-0 text-xs sm:text-sm">
        <p class="pt-2">无限图片储存数量，你可以上传不限数量的图片！</p>
        <p>图片首次访问后缓存，"永久"有效，包括全球分布的 CDN，以确保尽可能快地提供图像.</p>
        <p>图屋图床 是 <a class="text-slate-400" href="https://www.vvccc.cc" target="_blank" title="VC博客">VC博客</a> 支持并维护的文件上传项目，致力于为用户提供稳定的永久存储服务。</p>
      </AlertDescription>
    </Alert>

    <!-- 顶部广告（先空白，后期填图片链接） -->
    <div class="ad-top my-4 rounded-lg border border-dashed border-slate-300 p-3 bg-slate-50 text-center">
      <!-- 上线替换：href="广告链接" src="图片URL" -->
      <a href="" target="_blank">
        <img src="" alt="广告" class="max-w-full h-auto" />
      </a>
    </div>

    <!-- 工具栏 -->
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

    <!-- 上传 -->
    <Upload v-model="fileList" :UploadConfig="UploadConfig" :uploadAPI="uploadAPI" />
    <section v-show="fileList.length" class="vh-tools">
      <Button @click="fileList = []">清空</Button>
      <Button @click="vh.CopyText(fileList.map(item => item.upload_blob).join('\n'))">复制全部</Button>
    </section>

    <!-- 图片列表 -->
    <ResList v-model="fileList" :nodeHost="nodeHost" />

    <!-- 底部广告 -->
    <div class="ad-bottom mt-6 rounded-lg border border-dashed border-slate-300 p-3 bg-slate-50 text-center">
      <a href="" target="_blank">
        <img src="" alt="底部广告" class="max-w-full h-auto" />
      </a>
    </div>
  </section>
</template>

<script setup lang="ts">
import vh from 'vh-plugin'
import { ref, watch } from 'vue'
import { formatURL } from '@/utils/index'
import { Button } from '@/components/ui/button'
import Upload from '@/components/Upload/Upload.vue'
import ResList from '@/components/ResList/ResList.vue'
import { RocketIcon } from '@radix-icons/vue'
import { Label } from '@/components/ui/label'
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'
import { RadioGroup, RadioGroupItem } from '@/components/ui/radio-group'

const nodeHost = ref(import.meta.env.VITE_IMG_API_URL || location.origin)
const uploadAPI = ref(`${import.meta.env.VITE_IMG_API_URL || location.origin}/upload`)

const UploadConfig = ref({
  AcceptTypes: 'image/*',
  Max: 0,
  MaxSize: 15
})

const fileList = ref(JSON.parse(localStorage.getItem('zychUpImageList') || '[]'))

watch(fileList, (newVal) => {
  localStorage.setItem(
    'zychUpImageList',
    JSON.stringify(
      newVal
        .filter(item => item.upload_status === 'success')
        .map(item => {
          item.upload_blob = formatURL({ nodeHost: nodeHost.value }, item.upload_result)
          return item
        })
    )
  )
})
</script>

<style scoped lang="less">
@import 'Home.less';
.ad-top {
  min-height:70px;
}
.ad-bottom {
  min-height:70px;
}
</style>
