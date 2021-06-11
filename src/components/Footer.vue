<template>
  <footer class="relative max-w-screen-xl flex-1 mx-auto text-gray-500 justify-center flex flex-col h-14 items-center p-3 py-8 xl:px-0 px-3">
    <div class="absolute xl:left-0 left-3 flex items-center space-x-1  ">
      <i @click="setColorMode()" class="las la-lightbulb text-2xl dark:hover:text-oswapGreen dark:text-gray-500 text-oswapGreen cursor-pointer"></i>
      <p class="text-sm dark:text-oswapGreen mobile:hidden">{{colorMode}}</p>
    </div>
    <h1 class="text-sm dark:text-oswapGreen mobile:hidden mt-1">OpenSwap Exchange</h1>
    <div class="hidden mobile:flex mt-1">
      <Wallet/>
    </div>
    <div class="flex items-center space-x-3 mt-1">
      <a href="">
        <i class="lab la-twitter text-xl mobile:text-base hover:text-oswapGreen-dark dark:text-oswapGreen dark:hover:text-oswapGreen-light"></i>
      </a>
      <a href="">
        <i class="lab la-telegram text-xl mobile:text-base hover:text-oswapGreen-dark dark:text-oswapGreen dark:hover:text-oswapGreen-light"></i>
      </a>
      <a href="">
        <i class="lab la-instagram text-xl mobile:text-base hover:text-oswapGreen-dark dark:text-oswapGreen dark:hover:text-oswapGreen-light"></i>
      </a>
    </div>
    <div class="absolute hidden mobile:flex right-3 items-center ">
      <MoreButton/>
    </div>
  </footer>
</template>

<script>
import Wallet from '@/components/header/Wallet'
import MoreButton from '@/components/header/MoreButton'


export default {
  name: 'Footer',
  components: {
    Wallet,
    MoreButton
  },
  props: {
    colorMode: String
  },
  methods: {
    setColorMode() {
      this.$emit('dark-mode');
    }
  }
}
</script>