<template>
  <div id="farm" class="max-w-screen-xl mx-auto flex flex-1 flex-col items-center justify-center oswap-layout xl:px-0 px-3 text-gray-500 pb-16">
    <transition name="fall" appear>
      <FarmHeader :data="farmHeaderData" @updateData="getTotalPending"/>
    </transition>
    
    <transition name="farm" appear>
      <div v-if="soloData != null && farmData != null" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3 w-full">
        <SoloFarmPair  v-for="(pool, index) in this.SoloPools" @updateTVL="updateTVL" :key="index" :poolData="soloData[pool.i]" :pool="pool" @updateData="updateData"/>
        <FarmPair v-for="(pool, index) in this.Pools" @updateTVL="updateTVL" @updateAPR="updateAPR" :key="index" :poolData="farmData[pool.i]" :pool="pool" @updateData="updateData"/>
      </div>
      <div v-else class="flex h-full items-center mt-16">
        <svg class="animate-spin h-8 w-8 text-oswapGreen" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
      </div>
    </transition>
  </div>
</template>

<script>
  import FarmHeader from "@/components/farm/FarmHeader"
  import FarmPair from '@/components/farm/FarmPair'
  import SoloFarmPair from '@/components/farm/SoloFarmPair'

  import openswap from "@/shared/openswap.js";

  import { createWatcher } from '@makerdao/multicall';
  import { mapGetters, mapActions } from 'vuex';

  const { pools } = require("@/store/modules/farm/pools.js")

  export default {
    name: 'Farm',
    mixins: [openswap],
    components: {
      FarmHeader,
      FarmPair,
      SoloFarmPair
    },
    mounted: async function () {
      
      console.log(this.SoloPools)
      console.log(this.Pools)
      let timeout

      if(this.getUserSignedIn()){
        timeout = 1
      } else {
        timeout = 1000
      }

      await setTimeout(async function (){
        this.Pools = pools[this.getChainID()].pools;
      this.SoloPools = pools[this.getChainID()].SoloPools;
        this.farmData = await this.initMulticall(this.Pools)
        this.setFarmDataState(this.farmData);
        this.soloData = await this.initSoloMulticall(this.SoloPools)
        console.log(soloData)
        this.setSoloDataState(this.soloData);
        
      }.bind(this), timeout);

      let iid = await setInterval(async function(){
       
        await this.getTotalPending();
        this.farmData = await this.initMulticall(this.Pools)
        console.log(this.farmData)
        this.setFarmDataState(this.farmData);
        this.soloData = await this.initSoloMulticall(this.SoloPools)
        this.setSoloDataState(this.soloData);
        if(this.$route.name != 'OpenSwap Farms'){
          clearInterval(iid)
          console.log(this.$route.name)
        }
      }.bind(this), 15000)
      
    },
    data() {
      return {
        Pools: null,
        SoloPools: null,
        farmData: null,
        soloData: null,
        farmHeaderData: {
          rewardsPending: 0,
          TVL: 0,
          PVL: 0,
          APRs: {
            pAPR: 0,
            tAPR: 0
          },
          chartData: {
            name: '',
            liquidity: 0
          }
        },
      }
    },
    computed:{
      ...mapGetters('addressConstants', ['oSWAPMAKER', 'oSWAPCHEF', 'hMULTICALL', 'hRPC']),
    },
    methods: {
      ...mapGetters('wallet', ['getUserAddress', 'getUserSignedIn', 'getChainID']),
      ...mapActions('farm/farmData', ['setFarmDataState', 'setSoloDataState', 'setCustomDataState']),

      updateTVL: function(TVLData){
   
        this.farmHeaderData.TVL = this.farmHeaderData.TVL + TVLData.pool.TVL
        
        // sends info to chart
        this.farmHeaderData.chartData.name = TVLData.pool.name;
        this.farmHeaderData.chartData.liquidity = TVLData.pool.TVL;

        this.farmHeaderData.PVL = this.farmHeaderData.PVL + TVLData.pvl
      },

      updateAPR: function(APRData){
        if (APRData.staked > 0) {
          if (this.farmHeaderData.APRs.pAPR == 0) {
            this.farmHeaderData.APRs.pAPR = this.farmHeaderData.APRs.pAPR + parseFloat(APRData.pAPR)
          } else {
            this.farmHeaderData.APRs.pAPR = parseFloat((parseFloat(this.farmHeaderData.APRs.pAPR) + parseFloat(APRData.pAPR)) / 2).toFixed(1)
          } 
        } else if (this.farmHeaderData.APRs.tAPR == 0) {
          this.farmHeaderData.APRs.tAPR = this.farmHeaderData.APRs.tAPR + parseFloat(APRData.tAPR)
        } else {
          this.farmHeaderData.APRs.tAPR =  parseFloat((parseFloat(this.farmHeaderData.APRs.tAPR) + parseFloat(APRData.pAPR)) / 2).toFixed(1)
        }
      },

      updateData: async function(){
        this.farmData = await this.initMulticall(this.Pools)
        this.setFarmDataState(this.farmData);

        this.soloData = await this.initMulticall(this.SoloPools)
        this.setSoloDataState(this.soloData);
      },

      getTotalPending: async function(){
        let temp = 0;

        let userAddress = this.getUserAddress();
        const MASTERCHEF = this.oSWAPCHEF(this.getChainID());
        var i = 0;
        var poolByIndex = []
        var CALL = []
        for (var n in this.SoloPools) {
          //LP Balance CALLS
          CALL.push({
            target: MASTERCHEF,
            call: ['pendingSushi(uint256,address)(uint256)', parseInt(this.SoloPools[n].pid), userAddress],
            returns: [['PENDING_OF_' + n , val => val]]
          })
        }
        for (var n in this.Pools) {
          
          //LP Balance CALLS
          CALL.push({
            target: MASTERCHEF,
            call: ['pendingSushi(uint256,address)(uint256)', parseInt(this.Pools[n].pid), userAddress],
            returns: [['PENDING_OF_' + n , val => val]]
          })
        }
        

        const MULTICALL = this.hMULTICALL(this.getChainID());
        const RPC = this.hRPC(this.getChainID());
        
        var results= [];

        const config = {
          rpcUrl: RPC,
          multicallAddress: MULTICALL
        };

        const watcher = createWatcher(
            CALL,
            config
          );
        
        watcher.subscribe(update => { results.push(update) });
        watcher.start();
        await watcher.awaitInitialFetch();
  
        watcher.stop();

        for (var n in results) {
          temp = temp + parseFloat(this.getEthUnits(results[n].value.toString()))
        }

        this.farmHeaderData.rewardsPending = temp
      },
      initMulticall: async function(pools) {
        //const OPENMAKER = this.oSWAPMAKER();
        const MULTICALL = this.hMULTICALL(this.getChainID());
        const RPC = this.hRPC(this.getChainID());
        const [CALL, poolByIndex] = this.generateCalls(pools);
        var results= [];
        console.log(RPC)
        console.log(MULTICALL)
        const config = {
          rpcUrl: RPC,
          multicallAddress: MULTICALL
        };
        console.log(CALL)

        const watcher = createWatcher(
            CALL,
            config
          );
        
        watcher.subscribe(update => { results.push(update) });
        watcher.start();
        await watcher.awaitInitialFetch();

        var res = await this.parseResults(results, poolByIndex);
        watcher.stop();

        return res;
      },
      initSoloMulticall: async function(pools) {
        //const OPENMAKER = this.oSWAPMAKER();
        const MULTICALL = this.hMULTICALL(this.getChainID());
        const RPC = this.hRPC(this.getChainID());
        const [CALL, poolByIndex] = this.generateSoloCalls(pools);
        var results= [];

        const config = {
          rpcUrl: RPC,
          multicallAddress: MULTICALL
        };

        const watcher = createWatcher(
            CALL,
            config
          );
        
        watcher.subscribe(update => { results.push(update) });
        watcher.start();
        await watcher.awaitInitialFetch();

        var res = await this.parseSoloResults(results, poolByIndex);
        watcher.stop();
        return res;
      },
      parseResults: async function(results, poolByIndex){
        let count = results.length

        
        
        var dataObj = {}
        var numOfCallsPerPool = 8;
        var i,j, temporary
        var pid = 0
        for (i = 0,j = results.length; i < j; i += numOfCallsPerPool) {

          let farmData = {
          pool: null,
          lpBalance: null,
          lpBalanceStaked: null,
          lpStakedTotal: null,
          lpTokenTotalSupply: null,
          pendingReward: null,
          token0Pstaked: null,
          token1Pstaked: null,
          token0Tstaked: null,
          token1Tstaked: null,
          tvalue0: null,
          tvalue1: null
        }
            temporary = results.slice(i, i + numOfCallsPerPool);
            farmData.pool = poolByIndex[pid]
            farmData.lpBalance = temporary[0].value
            farmData.lpBalanceStaked = temporary[3].value
            farmData.lpStakedTotal = temporary[1].value
            farmData.lpTokenTotalSupply = temporary[4].value
            farmData.pendingReward = temporary[2].value
            farmData.token0Tstaked = temporary[5].value
            farmData.token1Tstaked = temporary[6].value
            farmData.stakeWeight = this.getStakeWeight(temporary[1].value,temporary[3].value)
            farmData.token0Pstaked = this.getPersonalAmounts(farmData.stakeWeight, farmData.token0Tstaked)
            farmData.token1Pstaked = this.getPersonalAmounts(farmData.stakeWeight, farmData.token1Tstaked)
         

            dataObj[pid] = farmData
            pid++
        }
        
        
        return dataObj;
        
     },
     getStakeWeight(amountStaked, totalStaked){
      let ps = parseFloat(this.getFormatedUnitsDecimals(amountStaked, 18))
     
      let ts = parseFloat(this.getFormatedUnitsDecimals(totalStaked, 18))

      if(ps == 0){
        return 0
      }

      return ts / ps * 100

     },
     getPersonalAmounts: function(weight, amount, decimals){
      weight = this.getBN(String((weight * 10000000).toFixed(0)))
      let hund = this.getBN('100')
      let mil = this.getBN('10000000')
   
      if(weight == 0){
        return 0
      }
      
      return amount.mul(weight).div(hund).div(mil)
     },
     parseSoloResults: async function(results, poolByIndex){
        let count = results.length

        
        
        var dataObj = {}
        var numOfCallsPerPool = 5;
        var i,j, temporary
        var pid = 0
        for (i = 0,j = results.length; i < j; i += numOfCallsPerPool) {

          let farmData = {
          pool: null,
          lpBalance: null,
          lpBalanceStaked: null,
          lpStakedTotal: null,
          lpTokenTotalSupply: null,
          pendingReward: null,
          token0Pstaked: null,
          token1Pstaked: null,
          token0Tstaked: null,
          token1Tstaked: null,
          tvalue0: null,
          tvalue1: null
        }
            temporary = results.slice(i, i + numOfCallsPerPool);
            farmData.pool = poolByIndex[pid]
            farmData.lpBalance = temporary[0].value
            farmData.lpBalanceStaked = temporary[3].value
            farmData.lpStakedTotal = temporary[1].value
            farmData.lpTokenTotalSupply = temporary[4].value
            farmData.pendingReward = temporary[2].value


            dataObj[pid] = farmData
            pid++
        }
        
        
        return dataObj;
        
     },
      generateCalls: function(pools){
        let CALL = [];
        let userAddress = this.getUserAddress();
        const MASTERCHEF = this.oSWAPCHEF(this.getChainID());
        console.log(MASTERCHEF)
        console.log("mass")
        var i = 0;
        var poolByIndex = []

        for (var n in pools) {
          poolByIndex[i] = pools[n]
          //LP Balance CALLS
          CALL.push(
            {
              target: pools[n].pairaddress,
              call: ['balanceOf(address)(uint256)', userAddress],
              returns: [['BALANCE_OF_' + n , val => val]]
            }
          );
          //Staked LP Balance Calls
          CALL.push(
            {
              target: pools[n].pairaddress,
              call: ['balanceOf(address)(uint256)', MASTERCHEF],
              returns: [['TOTAL_LP_STAKED_OF_' + n , val => val]]
            }
          );
          
          //unclaimed rewards calls
          CALL.push(
            {
              target: MASTERCHEF,
              call: ['pendingSushi(uint256,address)(uint256)', parseInt(pools[n].pid), userAddress],
              returns: [['PENDING_OF_' + n , val => val]]
            }
          ); 
          //total LP staked calls
          CALL.push(
            {
              target: MASTERCHEF,
              call: ['amountStaked(uint256,address)(uint256)', parseInt(pools[n].pid), userAddress],
              returns: [['LP_STAKED_OF_' + n , val => val ]]
            }
          );
          CALL.push(
            {
              target: pools[n].pairaddress,
              call: ['totalSupply()(uint256)'],
              returns: [['TOTAL_SUPPLY_OF_' + n , val => val]]
            }
          );
          CALL.push(
            {
              target: pools[n].pairaddress,
              call: ['getReserves()(uint256, uint256, uint256)'],
              returns: [['reserve0_' + n , val => val],['reserve1_' + n , val => val],['timestamp_ ' + n , val => val]]
            }
          );
          i++;
        }
        return [CALL, poolByIndex];
      },
      generateSoloCalls: function(pools){
        let CALL = [];
        let userAddress = this.getUserAddress();
        const MASTERCHEF = this.oSWAPCHEF(this.getChainID());
        var i = 0;
        var poolByIndex = []

        for (var n in pools) {
          poolByIndex[i] = pools[n]
          //LP Balance CALLS
          CALL.push(
            {
              target: pools[n].pairaddress,
              call: ['balanceOf(address)(uint256)', userAddress],
              returns: [['BALANCE_OF_' + n , val => val]]
            }
          );
          //Staked LP Balance Calls
          CALL.push(
            {
              target: pools[n].pairaddress,
              call: ['balanceOf(address)(uint256)', MASTERCHEF],
              returns: [['TOTAL_LP_STAKED_OF_' + n , val => val]]
            }
          );
          
          //unclaimed rewards calls
          CALL.push(
            {
              target: MASTERCHEF,
              call: ['pendingSushi(uint256,address)(uint256)', parseInt(pools[n].pid), userAddress],
              returns: [['PENDING_OF_' + n , val => val]]
            }
          ); 
          //total LP staked calls
          CALL.push(
            {
              target: MASTERCHEF,
              call: ['amountStaked(uint256,address)(uint256)', parseInt(pools[n].pid), userAddress],
              returns: [['LP_STAKED_OF_' + n , val => val ]]
            }
          );
          CALL.push(
            {
              target: pools[n].pairaddress,
              call: ['totalSupply()(uint256)'],
              returns: [['TOTAL_SUPPLY_OF_' + n , val => val]]
            }
          );
         
          i++;
        }
        return [CALL, poolByIndex];
      }
    }
    
  }
</script>