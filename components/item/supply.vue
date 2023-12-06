<template>
    <div class="dashboard-supply">
        <div style="padding: 5px; font-size: 14px; text-align: center; color:#324DFF">
            <b>지역별 설비보급량</b>
            <span style="float: right;"></span>
        </div>
        <div style="padding: 5px;">
            <table>
                <tbody>
                    <tr>
                        <!-- <td style="text-align: center; vertical-align: middle;">
                            <b style="font-size: 12px;">낮음</b>
                        </td>
                        <td style="padding: 0px;" >
                            <img src="../../assets/img/fourthGra.png" style="height: 12px; width: 100%; vertical-align: middle; position: relative; top: 1px;">
                        </td>
                        <td style="padding: 0px;">
                            <img src="../../assets/img/thirdGra.png" style="height: 12px; width: 100%; vertical-align: middle; position: relative; top: 1px;">
                        </td>
                        <td style="padding: 0px;">
                            <img src="../../assets/img/secondGra.png" style="height: 12px; width: 100%; vertical-align: middle; position: relative; top: 1px;">
                        </td>
                        <td style="padding: 0px;">
                            <img src="../../assets/img/firstGra.png" style="height: 12px; width: 100%; vertical-align: middle; position: relative; top: 1px;">
                        </td>
                        <td style="text-align: center; vertical-align: middle;">
                            <b style="font-size: 12px;">높음</b>
                        </td> -->
                        <td style="text-align: center; vertical-align: middle;">
                            <b style="font-size: 12px;">낮음</b>
                        </td>
                        <td style="padding: 0px; height: 12px;">
                            <div class="my_gradient_1"></div>
                        </td>
                        <td style="padding: 0px; height: 12px;">
                            <div class="my_gradient_2"></div>
                        </td>
                        <td style="padding: 0px; height: 12px;">
                            <div class="my_gradient_3"></div>
                        </td>
                        <td style="padding: 0px; height: 12px;">
                            <div class="my_gradient_4"></div>
                        </td>
                        <td style="text-align: center; vertical-align: middle;">
                            <b style="font-size: 12px;">높음</b>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>
  
<script>
export default {
name: 'supply',
components: {
},
data() {
    return{
        chartData:{},
        options:{}
    }
},

methods:{
},

computed:{
},

mounted(){
}
}
</script>