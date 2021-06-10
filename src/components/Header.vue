<template>
  <header class="flex  max-w-screen-xl mx-auto justify-between h-14 py-9 xl:px-0 px-3 items-center">
    
    <img alt="OpenSwap" src="../assets/oswap_logo.png" class="h-12">
    <!-- Navigation Start -->
    <div class="flex  items-center space-x-2 text-gray-500">
      <div class="flex flex-none mobile:hidden">
      <Routes />
      </div>

      <!-- Menu 'More' (dropdown) -->
       <div class="flex flex-none mobile:hidden">
      <MoreButton />
       </div>

      <!-- display oswap price -->
      <Oswap />

      <div class="flex flex-none mobile:hidden">
            <!-- Wallet Button  -->
            <Wallet/>
      </div>

    </div>
    <!-- Navigation End -->
  </header>
    <header class="flex-1 mx-auto py-2 xl:px-0 px-1 justify-center hidden mobile:flex">
    <div class="flex flex-wrap justify-center space-x-2 text-gray-500">
      <Routes />
    </div>
    <!-- Navigation End -->
  </header>
</template>

<script>
  import Wallet from '@/components/header/Wallet'
  import MoreButton from '@/components/header/MoreButton'

  import Oswap from '@/components/header/OswapPrice'
  import Routes from '@/components/header/Routes'
  import store from "../store";
  
  export default {
    name: 'Header',
    components: {
      Wallet,
      MoreButton,
      Oswap,
      Routes,
    },
    data() {
      return {        
      }
    },

    methods: {
    }
  }
</script>