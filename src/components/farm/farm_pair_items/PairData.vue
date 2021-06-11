<template>

  <p class="text-sm text-gray-600 leading-none mt-1 pt-3 dark:text-oswapGreen">LP Tokens Available: {{lpBalance}}</p>
  <p class="text-sm text-gray-600 leading-none mt-1 pt-3 dark:text-oswapGreen">Staked LP Tokens: {{staked}}</p>
  <p class="text-sm text-gray-600 leading-none mt-1 pt-3 dark:text-oswapGreen">Your Unclaimed Rewards: {{pendingSushi}}</p>
  <p class="text-sm text-gray-600 leading-none mt-1 pt-3 dark:text-oswapGreen">Stake Weight: {{poolWeight}}%</p>
  <p class="text-sm text-gray-600 leading-none mt-1 pt-3 dark:text-oswapGreen">{{poolOneName}} Staked: {{token0PStaked}}</p>
  <p class="text-sm text-gray-600 leading-none mt-1 pt-3 dark:text-oswapGreen">{{poolTwoName}} Staked: {{token1PStaked}}</p>
  <p class="text-sm text-gray-600 leading-none mt-1 pt-3 dark:text-oswapGreen">Expected Weekly rewards: $ {{weekly}} USD</p>
  <p class="text-sm text-gray-600 leading-none mt-1 pt-3 dark:text-oswapGreen">Expected Monthly rewards: $ {{monthly}} USD</p>
  <p class="text-sm text-gray-600 leading-none mt-1 pt-3 dark:text-oswapGreen">Total Staked: {{totalStakedUI}} {{poolOneName}}/{{poolTwoName}}</p>
  <p class="text-sm text-gray-600 leading-none mt-1 pt-3 dark:text-oswapGreen">Total {{poolOneName}} Staked: {{token0TStaked}}</p>
  <p class="text-sm text-gray-600 leading-none mt-1 pt-3 dark:text-oswapGreen">Total {{poolTwoName}} Staked:	{{token1TStaked}}</p>

</template>

<script>
  import openswap from "../../../shared/openswap.js";
  import { mapState } from 'vuex';

  export default {
    name: 'PairData',
    props: {
      poolOneName: String,
      poolTwoName: String,
      poolAddress: String
    },
    components: {
    },
    mixins: [openswap],
    computed: mapState('wallet', ['address']),
    watch: {
      address(newValue, oldValue) {
        if(oldValue != newValue){
          this.getLPBalance(this.poolAddress);
        }
      }
    },
    data() {
      return {
        lpBalance: 0,
        staked: 0,
        pendingSushi: 0,
        poolWeight: 0,
        token0PStaked: 0,
        token1PStaked: 0,
        weekly: 0,
        monthly: 0,
        totalStakedUI: null,
        token0TStaked: 0,
        token1TStaked: 0
      }
    },
    methods: {
    }
  }
</script>