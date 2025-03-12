<template>
  <Header v-if="!withoutHeader" />
  <article
    :class="{
      'mt-20': !withoutHeader,
      'mb-2': !withoutFooter,
    }"
  >
    <RouterView />
  </article>
  <Footer v-if="!withoutFooter" />
  <div
    id="loadingModal"
    :class="{
      'screen-animate': !loading,
    }"
    class="fixed flex overflow-hidden top-0 transition-all justify-center items-center h-full w-full left-0 bg-black-9 z-[10000]"
  >
    <img
      src="./common/images/header/logo@2x.webp"
      class="logo-animate text-7xl text-black w-[5rem] h-[2.875rem]"
      alt=""
    />
    <!-- <IconLogo
      :use-current-color="true"
      class="logo-animate text-7xl text-black"
    /> -->
  </div>
  <ConnectWallet />
  <Login />
</template>

<script setup lang="ts">
import { ref, watchEffect, onMounted } from 'vue'
import { RouterView, useRouter } from 'vue-router'
import FingerPrint2 from 'fingerprintjs2'
import { IconLogo } from '@arco-iconbox/vue-muverse-icon'

import { postVisiterLog } from '@/common/api/common'
import ConnectWallet from '@/components/connect-wallet'
import serverTime from '@/state/server-time'
import { getHomeInfo } from './common/api/home'

import Header from '@/components/header'
import Footer from '@/components/footer'
import Login from '@/components/login.vue'
import links from './state/applinks'

const { currentRoute } = useRouter()

const withoutFooter = ref(true)
const withoutHeader = ref(true)

const loading = ref(true)

watchEffect(() => {
  const { meta } = currentRoute.value
  if (meta.theme === 'black') {
    document.documentElement.classList.add('dark')
  } else {
    document.documentElement.classList.remove('dark')
  }
  withoutFooter.value = (meta.withoutFooter as boolean) || false
  withoutHeader.value = (meta.withoutHeader as boolean) || false
})

window.addEventListener('DOMContentLoaded', () => {
  setTimeout(() => {
    loading.value = false
    setTimeout(() => {
      const loadingModal = document.getElementById('loadingModal')
      if (loadingModal) {
        loadingModal.style.display = 'none'
      }
    }, 1000)
  }, 1000)
})

async function sendVisitLog() {
  const finger = localStorage.getItem('finger')
  if (finger) {
    await postVisiterLog({ device_id: finger })
  } else {
    FingerPrint2.get({}, async (components: any) => {
      // 参数
      const values = components.map(function (component: any) {
        return component.value
      })
      // 指纹
      const finger = FingerPrint2.x64hash128(values.join(''), 31)
      await postVisiterLog({ device_id: finger })
      localStorage.setItem('finger', finger) //存入标识值
    })
  }
}

onMounted(async () => {
  window.scrollTo({
    top: 0,
  })

  // const homeInfo = await getHomeInfo()
  // links.value = homeInfo?.linkList

  // serverTime.value = homeInfo.time
  setInterval(() => {
    serverTime.value += 1
  }, 1000)
  sendVisitLog()
})
</script>

<style lang="less">
.logo-animate {
  animation: LogoAnimation 1s infinite cubic-bezier(0.5, 0, 0.5, 1);
}

.screen-animate {
  animation: ScreenAnimation 1s 1 linear;
}

@keyframes LogoAnimation {
  50% {
    transform: scale(1.2);
  }
}

@keyframes ScreenAnimation {
  60% {
    opacity: 0.4;
    background-color: #fff;
  }
  100% {
    opacity: 0;
  }
}
</style>
