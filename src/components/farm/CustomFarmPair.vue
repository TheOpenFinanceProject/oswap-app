<template>
  <div class="flex flex-col p-3 st5 st5-all group bg-gradient-to-l dark:from-slightDark from-slightGray to-transparent dark:hover:bg-slightDark hover:bg-slightGray border-l border-oswapGreen rounded-3xl">
    <!-- Header -->
    <PoolHeader  :pool="pool" :poolData="poolData"/>
    <!-- Body -->
    <div class="flex flex-col h-full mt-3 relative">
      <!-- Show this when pool details is closed -->
      <PoolStatsClosed @setPool="setPool" :poolData="poolData" :pool="pool"  :isOpen="poolStatsOff" />

      <!-- Show this when pool details is opened -->
      <PoolStatsInfo :isOpen="poolStatsOn" :poolData="poolData" :pool="pool" @setPool="setPool" />

      <!-- Show this when the pool is opened and clicked on Stake -->
      <PoolStake :isOpen="poolStakeOn" :maxAmount="poolData.lpBalance" :pool="pool" @setPool="setPool" />

      <!-- Show this when the pool is opened and clicked on Unstake -->
      <PoolUnstake :isOpen="poolUnstakeOn" :maxAmount="poolData.lpBalanceStaked" :pool="pool" @setPool="setPool" />
    </div>
  </div>
</template>


<script>
  import openswap from "@/shared/openswap.js";

  import PoolHeader from "@/components/farm/CustomFarmPair/PoolHeader"
  import PoolStatsClosed from "@/components/farm/CustomFarmPair/PoolStatsClosed"
  import PoolStatsInfo from "@/components/farm/CustomFarmPair/PoolStatsInfo"
  import PoolStake from "@/components/farm/CustomFarmPair/PoolStake"
  import PoolUnstake from "@/components/farm/CustomFarmPair/PoolUnstake"

  export default {
    name: 'CustomFarmPair',
    props: {
      pool: Object,
      poolData: Object,
    },
    components: {
      PoolHeader,
      PoolStatsClosed,
      PoolStatsInfo,
      PoolStake,
      PoolUnstake
    },
    mixins: [openswap],
    mounted: async function (){
      
    },
    data() {
      return {
        //pool: null,
        poolStatsOff: true,
        poolStatsOn: false,
        poolStakeOn: false,
        poolUnstakeOn: false,
      }
    },
    methods: {
      setPool(value) {
        if (value == 'open') {
          this.$el.classList.add('row-span-3', 'ensure-height', 'ring-1', 'ring-inset', 'ring-oswapGreen');
          this.poolStatsOff = false
          this.poolStatsOn = true
        }

        if (value == 'close') {
          this.$el.classList.remove('row-span-3', 'ensure-height', 'ring-1', 'ring-inset', 'ring-oswapGreen');
          this.poolStatsOn = false
          this.poolStatsOff = true
        }

        if (value == 'stake') {
          this.poolStatsOn = false
          this.poolStakeOn = true
        }

        if (value == 'stats') {
          this.poolStakeOn = false
          this.poolUnstakeOn = false
          this.poolStatsOn = true
        }

        if (value == 'unstake') {
          this.poolStatsOn = false
          this.poolUnstakeOn = true
        }
      }
    }
  }
</script>

<style lang="scss" scoped>
  .ensure-height {
    height: 420px;
  }
</style>