<template>
    <div style="height: 220px; width: 260px; font-size: 14px; font-weight: bold;">
        지역별 태양광 발전량
        <Bar :chart-data="chartData" :chart-options="options" :styles="styles"/>
    </div>
</template>
  
<script lang="ts">
import {
Chart as ChartJS,
Title,
Tooltip,
Legend,
BarElement,
CategoryScale,
LinearScale,
PointElement,
LineElement,
} from 'chart.js'
import { Bar } from 'vue-chartjs'

ChartJS.register(CategoryScale, LinearScale, BarElement, PointElement, LineElement, Title, Tooltip, Legend)

export default {
  name: 'App',
  components: {
   Bar
  },
  data() {
   return{
    chartData:{},
    options:{},
    bjdongCdList: [],
    genList: [],
    prevGenList: [],
   }
  },

  props: ['genByRegionList'],

  methods:{
   setData(){
    let tempLabels = [];    
    let tempGenData = [];
    let tempPrevGenData = [];

    for(let i=0; i<this.bjdongCdList.length; i++){
        tempLabels.push(this.bjdongCdList[i].bjdongNm);
    }

    for(let i=0; i<this.genList.length; i++){
        tempGenData.push(this.genList[i].curGen); //발전량
    }

    for(let i=0; i<this.prevGenList.length; i++){
        tempPrevGenData.push(this.prevGenList[i].curGen); //전일 발전량
    }

    let tempData = {
        labels: tempLabels,
        datasets: [
        {
            label: '이번달',
            backgroundColor: '#5266EA',
            data: tempGenData,
            type :'bar',
            borderRadius: 15,
            barThickness: 6,
            order: 2, //위에 올라올 순서
        },
        {
            label: '지난달',
            backgroundColor: '#FD955E',
            borderColor: '#FD955E',
            borderWidth: 1,
            pointBackgroundColor: 'white',
            data: tempPrevGenData,
            type :'line',
            order: 1, //위에 올라올 순서
        }
        ]
    }

    let tempOptions = {
     // responsive: false,
     maintainAspectRatio: false,
     plugins: {
        legend: {
            display: true, //밤례 상태
            position: "bottom",
            labels:{
                boxWidth: 14,
                boxHeight: 5,
                font:{
                    size: 9
                }
            }
        },
        datalabels:{
            display: false
        }
     },
     layout: {
        padding: {
            top: 20,
            bottom: 5
        }
     },
     scales: {
        x: {
            ticks: {
                font: {
                    size: 9,
                }
            },
            grid: {
                display: false
            }
        },
        y: {
            ticks: {
                font: {
                    size: 9,
                }
            },
            grid: {
                display: false
            }
        }
     }
    }
    this.chartData = tempData;
    this.options = tempOptions;
   },
   
   //w값 => kw값 or kw => mw 변경
   fn_roundWToKw(value){
      return Math.round(value / 10000) / 100;    // / 1000   /1000
    },
  },

  computed:{
    styles(){
        return{
            height: '100%',
            position: 'relative'
        }
    }
  },

  mounted(){
  },

  watch:{
    genByRegionList(){
      this.bjdongCdList = this.genByRegionList.bjdongCdList;
      this.genList = this.genByRegionList.genList;
      this.prevGenList = this.genByRegionList.prevGenList;
      this.setData();
    }
  }
}
</script>