<template>
  <div class="flex flex-1 flex-col ss:w-80 xs:w-96 pb-5 pt-2.5">
    <div class="flex flex-1 flex-col p-4 st5 text-gray-500 dark:text-gray-300 bg-gradient-to-l from-slightGray dark:from-slightDark to-transparent border-l border-oswapGreen rounded-3xl h-auto">
      <div class="flex items-center space-x-2 mb-3">
        <i class="las la-tint text-xl"></i>
        <p class="text-sm">Liquidity</p>
      </div>
      <div class="flex flex-col st5 dark:bg-oswapDark-gray bg-gray-100 rounded-2xl">
        <div class="flex shadow-lg flex-col st5 dark:bg-oswapDark-gray rounded-2xl">
          <div class="flex shadow-lg flex-col st5 dark:bg-oswapDark-gray rounded-2xl">
            <div class="flex shadow-lg flex-col space-y-3 st5 dark:bg-oswapDark-gray p-3 rounded-2xl">
              <LiquidityPair :available="balances.lpToken"/>
            </div>
            <LiquidityInfo :pairAddress="pairAddress"/>
          </div>
          <button @click="toggleAdd" class="flex px-3 py-4 space-x-3 items-center focus:outline-none">
            <i class="las la-plus-circle text-lg text-oswapGreen"></i>
            <p class="text-sm">Add Liquidity</p>
          </button>
          <transition tag="div" name="squeeze-liq" class="flex overflow-hidden">
            <div v-if="addLiquidity" ><AddLiquidity :key="createNewPair" :createNewPair="createNewPair" :balances="balances" @setSlippageRate="setSlippage" /></div>
          </transition>
        </div>
        <button @click="toggleRemove" class="flex px-3 py-4 space-x-3 items-center focus:outline-none">
          <i class="las la-minus-circle text-lg text-red-300"></i>
          <p class="text-sm">Remove Liquidity</p>
        </button>
        <transition tag="div" name="squeeze-liq" class="flex overflow-hidden">
          <div v-if="removeLiquidity" ><RemoveLiquidity :balances="balances" /></div>
        </transition>
      </div>
      

      <div class="flex pt-4">
        <div class="flex w-full h-10 items-center">
          <LiquidityBackButton />
            <div v-if="loaded" class="flex flex-1 h-full space-x-2 justify-end">
              <div v-if="addLiquidity" class="flex items-center w-28 h-full relative">
                <LiquidityApproveButton v-if="!token0Approved" :amount="getToken0Amount()" :token="getToken()['token1']" @set0approved="set0approved" />
                <LiquidityApproveButton v-if="token0Approved" :amount="getToken1Amount()" :token="getToken()['token2']" @set0approved="set0approved" :token0Approved="token0Approved" />
              </div>

              <div v-if="removeLiquidity" class="flex items-center w-28 h-full relative">
                <LiquidityApproveButton :amount="amount" :token="pairToken" @set0approved="set0approved" />
              </div>

              <div v-if="addLiquidity" class="flex items-center w-28 h-full relative">
                <LiquidityAddButton @executeAddLiquidity="executeAddLiquidity"/>
              </div>

              <div v-if="removeLiquidity" class="flex items-center w-28 h-full relative">
                <LiquidityRemoveButton @executeRemoveLiquidity="executeRemoveLiquidity"/>
              </div>
            </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
  import LiquidityPair from '@/components/liquidity/LiquidityPair';
  import LiquidityInfo from '@/components/liquidity/LiquidityInfo';
  import AddLiquidity from '@/components/liquidity/AddLiquidity';
  import RemoveLiquidity from '@/components/liquidity/RemoveLiquidity';
  import LiquidityBackButton from '@/components/liquidity/buttons/LiquidityBackButton';
  import LiquidityApproveButton from '@/components/liquidity/buttons/LiquidityApproveButton';
  import LiquidityAddButton from '@/components/liquidity/buttons/LiquidityAddButton';
  import LiquidityRemoveButton from '@/components/liquidity/buttons/LiquidityRemoveButton';

  import { mapGetters, mapActions } from 'vuex'
  import openswap from "@/shared/openswap.js";
  import { ethers } from 'ethers'
  import { createWatcher } from '@makerdao/multicall';

  export default {
    name: 'AddRemoveLiquidity',
    mixins: [openswap],
    components: {
      LiquidityPair,
      LiquidityInfo,
      AddLiquidity,
      RemoveLiquidity,
      LiquidityBackButton,
      LiquidityApproveButton,
      LiquidityAddButton,
      LiquidityRemoveButton
    },
    data() {
      return {
        amount: '1',
        loaded: false,
        token0Approved: false,
        token1Approved: false,
        slippageRate: '0.5',
        warnings: {},
        addLiquidity: false,
        removeLiquidity: false,
        createNewPair: false,
        pairAddress: null,
        pairToken: null,
        balances: {
          token0: 0,
          token1: 0,
          lpToken: 0
        }
      }
    },
    mounted: async function(){
      this.setInputAmount( {0: 0})
      this.setInputAmount({1: 0})
     
      this.pair = await this.getPair(this.getToken()['token1'],this.getToken()['token2'])
      .catch((err) => {
        console.log(err)
        this.createNewPair = true
      })
      if(!this.createNewPair){
        this.pairAddress = this.pair["liquidityToken"].address;
        this.pairToken = await this.getPairAsToken(this.getToken()['token1'],this.getToken()['token2'])
        await this.initMulticall()

      }
      this.loaded = true;
      
      
      
      
    },
    computed: {
      ...mapGetters('addressConstants', ['hMULTICALL', 'hRPC', 'WONE']),
    },
    methods: {
      ...mapGetters('wallet', ['getChainID']),
      ...mapGetters('exchange', ['getToken']),
      ...mapGetters('liquidity/amounts', ['getToken0Amount','getToken1Amount']),
      ...mapActions('exchange/swapper', [ 'setInputAmount']),
      ...mapActions('liquidity/buttons', ['setBtnState']),
      executeRemoveLiquidity:async function(){
        this.setBtnState({remove: 'removing'})
        let token0 = this.getToken()['token1']
        let token1 = this.getToken()['token2']
        let amount0 = this.getToken0Amount()

        await this.removeLiquidityParse(token0, token1, amount0, this.slippageRate)
        await this.initMulticall()
        
      },
      executeAddLiquidity: async function(){
        this.setBtnState({add: 'adding'})
        let token0 = this.getToken()['token1']
        let token1 = this.getToken()['token2']
        let amount0 = this.getToken0Amount()
        let amount1 = this.getToken1Amount()
        await this.addLiquidityParse(token0, token1, amount0, amount1, this.slippageRate)
        await this.initMulticall()
        
      },
      setSlippage(value){
        this.slippageRate = value;
      },
      set0approved(){
        this.token0Approved = true
      },
      set1approved(){
        this.token1Approved = true
      },
      parseResults: async function(results){
        const token0 = this.getToken()['token1']
        const token1 = this.getToken()['token2']


        this.balances.token0 = this.getFormatedUnits(results[1]['value'].toString(), token0)
        this.balances.token1 = this.getFormatedUnits(results[2]['value'].toString(), token1)
        this.balances.lpToken = this.getEthUnits(results[0]['value'])
        
        if(token0.oneZeroxAddress == this.WONE(this.getChainID())){
          this.balances.token0 = this.getEthUnits(await this.getOneBalance())
        }
        if(token1.oneZeroxAddress == this.WONE(this.getChainID())){
          this.balances.token1 = this.getEthUnits(await this.getOneBalance())
        }
     },
      initMulticall: async function(){
        const MULTICALL = this.hMULTICALL(this.getChainID());
        const RPC = this.hRPC(this.getChainID());
        const CALL = this.generateCalls();
        var results= [];
        console.log(CALL)
        console.log(MULTICALL)
        console.log(RPC)
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
        var res = await this.parseResults(results);
        return res;
      },
      generateCalls: function(){
        let CALL = [];
        let userAddress = this.getUserAddress();
        let token0 = this.getToken()['token1'].oneZeroxAddress
        let token1 = this.getToken()['token2'].oneZeroxAddress
        

          //LP Balance CALLS
          CALL.push(
            {
              target: this.pairAddress,
              call: ['balanceOf(address)(uint256)', userAddress],
              returns: [['BALANCE_OF_LP', val => val]]
            }
          );
          //Staked LP Balance Calls
          CALL.push(
            {
              target: token0,
              call: ['balanceOf(address)(uint256)', userAddress],
              returns: [['BALANCE_OF_T0', val => val]]
            }
          );
          
          //unclaimed rewards calls
          CALL.push(
            {
              target: token1,
              call: ['balanceOf(address)(uint256)', userAddress],
              returns: [['BALANCE_OF_T1', val => val]]
            }
          ); 
        return CALL;
      },
    

      toggleAdd() {
        this.addLiquidity = true;
        this.removeLiquidity = false;
      },

      toggleRemove() {
        this.removeLiquidity = true;
        this.addLiquidity = false;
      },

      setAmountOut(value){},

      setPriceImpact(value){
        this.priceImpact = value;
        if (this.priceImpact > 3) { 
          this.warnings['impact'] = 'Price impact high. Check reserves. Continue only if you know what you are doing.'
        } else { delete this.warnings['impact'] }
      },

      setAmount(value) {
        this.amount = value;
        if (this.amount !== '' && this.getBtnState({approve: 'approved'})) {
          //this.setBtnState({swap: 'swap'})
        }
      },

      setPath(value){
        this.path = value;
      },

      setSlippageRate(value){
        this.slippageRate = value;
      },
    }
  }
</script>