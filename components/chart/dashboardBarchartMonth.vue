<template>
    <div style="height: 220px; width: 260px; font-size: 14px; font-weight: bold;">
        <div class="switch-wrapper">
            <input id="gen" @change="typeChange($event)" value="gen" type="radio" name="switch" checked>
            <input id="use" @change="typeChange($event)" value="use" type="radio" name="switch">
            <label for="gen">태양광 발전량 추이</label>
            <label for="use">사용량 추이</label>
            <span class="highlighter"></span>
        </div>
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
data: () => ({
    chartData:{},
    options:{},
    type: "gen",
    currentGenList: [],
    prevGenList: [],
    currentUseList: [],
    prevUseList: [],
}),

props: ['genList', 'useList'],

methods:{
    setData(){
        let currentList = [];
        let prevList = [];
        let currentDataList = []; //올해 데이터
        let prevDataList = []; //작년 데이터
        let nodataFlag = true;
        let currentDt = null;
        let prevDt = null;

        if(this.type === "gen"){
            currentList = this.currentGenList;
            prevList = this.prevGenList;
        }else{
            currentList = this.currentUseList;
            prevList = this.prevUseList;
        }

        if(currentList.length > 0){
            for(let i=0; i<12; i++){
                for(let j=0; j<currentList.length; j++){
                    if(currentList[j].stDt === null){
                        currentDt = currentList[j].compDt
                    }else{
                        currentDt = currentList[j].stDt
                    }

                    if((i + 1) === Number(currentDt.substr(4))){
                        currentDataList.push(currentList[j].curValue);
                        nodataFlag = false;
                        break;
                    }else{
                        nodataFlag = true;
                    }
                }
                if(nodataFlag){
                    currentDataList.push(0)
                }
            }
        }else{
            for(let i=0; i<12; i++){
                currentDataList.push(0)
            }
        }

        if(prevList.length > 0){
            for(let i=0; i<12; i++){
                for(let j=0; j<prevList.length; j++){
                    if(prevList[j].stDt === null){
                        prevDt = prevList[j].compDt
                    }else{
                        prevDt = prevList[j].stDt
                    }

                    if((i + 1) === Number(prevDt.substr(4))){
                        prevDataList.push(prevList[j].curValue)
                        nodataFlag = false;
                        break;
                    }else{
                        nodataFlag = true;
                    }
                }
                if(nodataFlag){
                    prevDataList.push(0)
                }
            }
        }else{
            for(let i=0; i<12; i++){
                prevDataList.push(0)
            }
        }

        let tempData = {
            labels: [
                '1',
                '2',
                '3',
                '4',
                '5',
                '6',
                '7',
                '8',
                '9',
                '10',
                '11',
                '12'
            ],
            datasets: [
                {
                    label: '올해',
                    backgroundColor: '#5266EA',
                    data: currentDataList,
                    type :'bar',
                    borderRadius: 15,
                    barThickness: 6,
                    order: 2, //위에 올라올 순서
                },
                {
                    label: '작년',
                    backgroundColor: '#FD955E',
                    borderColor: '#FD955E',
                    borderWidth: 1,
                    pointBackgroundColor: 'white',
                    data: prevDataList,
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

    async typeChange(e){
        this.type = e.target.value;
        this.setData();
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

watch: {
    genList(){
        this.currentGenList = this.genList.genList;
        this.prevGenList = this.genList.preGenList;
        this.setData();
    },

    useList(){
        this.currentUseList = this.useList.useList;
        this.prevUseList = this.useList.preUseList;
        this.setData();
    },
},

mounted(){
    }
}
</script>