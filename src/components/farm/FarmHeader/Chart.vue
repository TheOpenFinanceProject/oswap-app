<template>
  <div class="flex flex-none items-center relative">
    <div class="flex z-30">
      <apexchart type="donut" width="200" height="224" :options="chartOptions" :series="series"></apexchart>
    </div>
    <div class="flex flex-col z-20 items-center text-gray-500 dark:text-gray-300 center-component">
      <p class="ss:text-5xl xs:text-7xl">{{totalPools}}</p>
      <p class="text-xs">Farm Pools</p>
    </div>
  </div>
</template>

<script>
  import VueApexCharts from "vue3-apexcharts";
  import { ethers } from 'ethers'; 

  export default {
    name: 'Chart',
    components: {
      apexchart: VueApexCharts
    },
    props: {
      poolName: String,
      liquidity: Number
    },
    watch: {
      poolName(value) {        
        this.series.push(this.liquidity);
        this.chartOptions.labels.push(this.poolName);
      },
    },
    methods: {
      prettify: function(number){
        return  ethers.utils.commify(number)
      }
    },
    data() {
      return {
        totalPools: 16,
        series: [],
        chartOptions: {
          chart: {
            type: 'donut'
          },
          responsive: [{
            breakpoint: 540,
            options: {
              chart: {
                width: 150,
                height: 168
              }
            }
          }],
          labels: [],
          plotOptions: {
            pie: {
              expandOnClick: false,
              donut: {
                size: '85%'
              }
            }
          },
          colors: [
            '#1bf2ba', '#18d5bb', '#13b4ba', '#109dbb',
            '#1bf2ba', '#18d5bb', '#13b4ba', '#109dbb',
            '#1bf2ba', '#18d5bb', '#13b4ba', '#109dbb',
            '#1bf2ba', '#18d5bb', '#13b4ba', '#109dbb',
            '#1bf2ba', '#18d5bb', '#13b4ba', '#109dbb',
            '#1bf2ba', '#18d5bb', '#13b4ba', '#109dbb'
          ],
          dataLabels: {
            enabled: false
          },
          legend: {
            show: false,
          },
          stroke: {
            show: false,
            curve: 'smooth',
            lineCap: 'round',
            colors: 'rgba(49, 53, 71, 1)',
            width: 1,      
          },
          states: {
            normal: {
              filter: {
                type: 'none',
                value: 0,
              }
            },
            hover: {
              filter: {
                type: 'none',
                value: 0.15,
              }
            },
            active: {
              allowMultipleDataPointsSelection: false,
              filter: {
                type: 'none',
                value: 0.35,
              }
            },
          },
          tooltip: {
            theme: false,
            custom: ({ series, seriesIndex, dataPointIndex, w }) => {
              return (
                '<div class="flex p-3 rounded-full bg-gray-100 dark:bg-slightDark text-xs text-gray-500 dark:text-gray-200 border-2 border-oswapGreen">' +
                "<span>" + w.globals.labels[seriesIndex] + ": " + "$ " + this.prettify(parseFloat(series[seriesIndex]).toFixed(2)) + "</span>" +
                "</div>"
              );
            }
          }
        },
      };
    },
  }
</script>