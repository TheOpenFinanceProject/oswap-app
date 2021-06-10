<template>
      <template  v-for="(route, index) in routes" :key="index">
        <router-link  :to="route.url" class="flex items-center space-x-2 p-2 px-3 rounded-lg group focus:outline-none focus:ring-1 focus:ring-black focus:ring-opacity-20">
          <i :class="'las text-xl dark:group-hover:text-oswapBlue-light group-hover:text-oswapGreen dark:text-oswapGreen '+route.icon"></i>
          <p class="text-sm group-hover:text-oswapBlue-light dark:text-oswapGreen">{{route.description}}</p>
        </router-link>
      </template >
</template> 

<script>
  
  export default {
    name: 'Routes',
    components: {
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