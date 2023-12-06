<template>
  <div>
    <div style="font-size: 14px; font-weight: bold;">발전소운영현황</div>
    <div style="height: 200px; width: 260px;">
      <Doughnut :chartData="chartData" :chartOptions="options" :plugins="[plugins]" :styles="myStyles" #customDoughnut/>
    </div>
  </div>
</template>

<script>
import { Chart as ChartJS, ArcElement, Tooltip, Legend } from 'chart.js'
import { Doughnut } from 'vue-chartjs'

ChartJS.register(ArcElement, Tooltip, Legend)

export default {
  name: 'doughnut',
  components: {
    Doughnut
  },
  data() {
    return{
      chartData:{},
      options:{},
      plugins:{},
      myStyles:{
        height: '100%',
        width: '100%',
        position: 'relative',
        top: '15px',
      }
    }
  },
  props: ['powerPlantList'],

  methods:{
    setData(){
      let resultData = [];
      let resultLabels = [];
      let resultBackGroundColor = [];
      let noDataFlag = true;

      for(let i=0; i<this.powerPlantList.length; i++){
        resultData.push(this.powerPlantList[i].count);
        if(this.powerPlantList[i].count !== 0){
          noDataFlag = false;
        }
      }
      

      if(!noDataFlag){
        resultLabels = ['정상', '경고', '고장', '준비중', '미연동']
        resultBackGroundColor = ['#324DFF', '#FFC56E', '#FF9069', '#9395A1', '#CACCD6']
      }else{
        // nodata test
        
        // resultLabels = ['정상', '경고', '고장', '준비중', '미연동']
        // resultBackGroundColor = ['#324DFF', '#FFC56E', '#FF9069', '#9395A1', '#CACCD6']
        // resultData = [ 861, 62, 9, 0, 0 ]

        resultData = [ 1 ]
        resultLabels = ['No Data'];
        resultBackGroundColor = ['#ededed'];
      }

      let tempData = {
        labels: resultLabels,
        datasets: [
          {
            backgroundColor: resultBackGroundColor,
            data: resultData,
            pointStyle: 'Rounded',
            pointRadius: 1,
            hoverBorderWidth: 5,
            hoverBorderColor: (context) => {
              return resultBackGroundColor[context.dataIndex];
            },
          }
        ]
      }

      let tempOptions = {
        // responsive: true,
        maintainAspectRatio: false,
        height: 300,
        plugins: {
          legend: {
            position: "bottom",
            title: { display: true, padding: 3 }, // or whatever number
            labels:{
              usePointStyle: true,
              font:{
                size: 9
              },
              // padding: 30
              // generateLabels: function(chart) {
              //   var data = chart.data;
              //   console.log(data)
              //   if (data.labels.length && data.datasets.length) {
              //     return data.labels.map(function(label, i) {
              //         return {
              //             text: label,
              //             fillStyle: data.datasets[0].backgroundColor[i],
              //             hidden: i === 1, // Set the second item to be initially disabled
              //             // Extra properties for you to add custom logic
              //             index: i,
              //         };
              //     });
              //   }
              //   return [];
              // }
            }
          },
          tooltip :{
            enabled: resultData.length === 1 ? false : true
          },
          datalabels: { 
            backgroundColor: function(context) {
              return '#FFFFFF';
            },
            borderColor: function(context){
              let index = context.dataIndex;
              return context.dataset.backgroundColor[index]
            },
            borderRadius: 25,
            borderWidth: 1,
            color: function(context){
              // let index = context.dataIndex;
              // return context.dataset.backgroundColor[index]
              return '#26283D'
            },
            display: true, //겹쳐지면 낮은 값 안보이게 (true/'auto')
            anchor: 'end',
            font: {
              size :11,
              weight: 'bold',
            },
            padding: {
              right: 8,
              left: 8,
              top: 6,
              bottom: 6
            },
            textAlign: 'center',
            formatter: (value, ctx) => {
              let label = "";
              let point = "●";
              if(resultData.length === 1){
                return null
              }else{
                if(value > 0){
                  label = ctx.chart.data.labels[ctx.dataIndex];
                  return label + '\n' + value; // Customize label format
                }else{
                  return null
                }
              }
            },
          }
        },
        layout: {
          padding: {
            top: 20,  //set that fits the best
          }
        },
      }

      let tempPlugins = {
        beforeInit(chart) {
          // reference of original fit function
          const originalFit = chart.legend.fit;
          // override the fit function
          chart.legend.fit = function fit() {
            // call original function and bind scope in order to use `this` correctly inside it
            originalFit.bind(chart.legend)();
            // increase the width to add more space
            this.height += 10;
          }; 
        },
      }

      this.chartData = tempData;
      this.options = tempOptions;
      this.plugins = tempPlugins;
    },
  },

  mounted(){
  },

  watch:{
    powerPlantList(){
      this.setData();
    }
  }
}
</script>

<style>
#customDoughnut{
  width:100%;
  height: 100%;
  padding: 10px;
  padding-bottom: 120px;
}
</style>