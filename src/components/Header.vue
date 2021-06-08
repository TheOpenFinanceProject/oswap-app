<template>
  <header class="max-w-screen-xl mx-auto flex justify-between h-14 py-9 xl:px-0 px-3 items-center">
    <img alt="OpenSwap" src="../assets/oswap_logo.png" class="h-12">
    <!-- Navigation Start -->
    <div class="flex items-center space-x-2 text-gray-500">
      
      <template  v-for="(route, index) in routes" :key="index">
        <router-link  :to="route.url" class="flex items-center space-x-2 p-2 px-3 rounded-lg group focus:outline-none focus:ring-1 focus:ring-black focus:ring-opacity-20">
          <i :class="'las text-xl dark:group-hover:text-oswapBlue-light group-hover:text-oswapGreen dark:text-oswapGreen '+route.icon"></i>
          <p class="text-sm group-hover:text-oswapBlue-light dark:text-oswapGreen">{{route.description}}</p>
        </router-link>
      </template >
      <!-- Menu 'More' (dropdown) -->
      <MoreButton />


      <!-- display oswap price -->
      <Oswap />


      <!-- Wallet Button  -->
      <Wallet/>

    </div>
    <!-- Navigation End -->
  </header>
</template>

<script>
  import Wallet from '@/components/header/Wallet'
  import MoreButton from '@/components/header/MoreButton'

  import Oswap from '@/components/header/OswapPrice'
  import store from "../store";
  
  export default {
    name: 'Header',
    components: {
      Wallet,
      MoreButton,
      Oswap
    },
    data() {
      return {        
        routes:[
          {
            description: 'Exchange',
            url: '/',
            icon: 'la-random'
          },
          {
            description: 'Liquidity',
            url: '/liquidity',
            icon: 'la-tint'
          },
                    {
            description: 'Farm',
            url: '/farm',
            icon: 'la-tractor'
          }
        ]
      }
    },

    methods: {
    }
  }
</script>