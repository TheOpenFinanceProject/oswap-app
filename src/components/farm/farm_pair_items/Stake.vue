<template>

	<button mat-icon-button="" class="flex-no-shrink bg-oswapGreen dark:bg-oswapDark-gray px-5 ml-4 py-2 text-sm shadow-sm hover:shadow-lg font-medium tracking-wider border-2 border-oswapGreen dark:text-oswapGreen text-white rounded-full" @click="this.collectAll()">Stake</button> 

</template>

<script>
  import openswap from "../../../shared/openswap.js";

  export default {
    name: 'Stake',
    props: {
      pool: Object
    },
    components: {
    },
    mixins: [openswap],
    data() {
      return {
      }
    },
    methods: {
    }
  }
</script>