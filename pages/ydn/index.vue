<template>
    <div class="dashboard-map" :style="customWindow">
      <div id="map" style="height:100%;">
        <v-select
          v-model="year"
          :items="years"
          style="position:absolute; z-index: 1; border-radius: 10px; width: 140px; margin-left: 20px; margin-top: 20px; padding: 0; background-color: aliceblue;"
          class="centered-selected-input"
          hide-details
          solo
        ></v-select>
        <v-select
          v-model="month"
          :items="months"
          style="position:absolute; z-index: 1; border-radius: 10px; width: 110px; margin-left: 170px; margin-top: 20px; padding: 0; background-color: aliceblue;"
          class="centered-selected-input"
          hide-details
          solo
        ></v-select>
        <div class="dashboard-status">
          <table>
            <colgroup>
              <col>
              <col>
            </colgroup>
            <tbody>
              <!-- 렌더링 안되는 버그로 임시 -->
              <tr style="display: none;">
                <td>발전소 수</td>
                <td class="count-num" start-count="0" target="siteCnt">0</td>
              </tr>
              <tr>
                <td>발전소 수</td>
                <td class="count-num" start-count="0" target="siteCnt" unit="개" style="color: #FFB800; padding-top: 20px;" >0&nbsp;<span>개</span></td>
              </tr>
              <tr>
                <td>설비용량</td>
                <td class="count-num" start-count="0" target="equipSize" unit="MW">0&nbsp;<span>MW</span></td>
              </tr>
              <tr>
                <td>RE달성률</td>
                <td class="count-num" start-count="0" target="energyindi" unit="%">0&nbsp;<span>%</span></td>
              </tr>
              <tr>
                <td>발전량</td>
                <td class="count-num" start-count="0" target="totalCurGen" unit="MWh">0&nbsp;<span>MWh</span></td>
              </tr>
              <tr>
                <td>소비량</td>
                <td class="count-num" start-count="0" target="totalCurUsg" unit="MWh">0&nbsp;<span>MWh</span></td>
              </tr>
              <tr>
                <td>탄소 저감량</td>
                <td class="count-num" start-count="0" target="carbonReduction" unit="tCO">0&nbsp;<span>tCO<sub>2</sub></span></td>
              </tr>
              <tr>
                <td>식수 효과</td>
                <td class="count-num" start-count="0" target="plantingEffect" unit="그루" style="padding-bottom: 20px;">0&nbsp;<span>그루</span></td>
              </tr>
            </tbody>
          </table>
        </div>
        <supply/>
        <div class="dashboard-doughnut">
          <doughnut :powerPlantList="powerPlantList"/>
        </div>
        <div class="dashboard-barchart-day">
          <dashboardBarchartDay :genByRegionList="genByRegionList"/>
        </div>
        <div class="dashboard-barchart-month">
          <dashboardBarchartMonth :genList="genList" :useList="useList"/>
        </div>
      </div>
    </div>
</template>

<script>
import { simcheonmyeon, yangsanmyeon, haksanmyeon, yonghawmyeon, yanggangmyeon, sangchonmyeon, maegokmyeon, chupungnyeongmyeon, hwangganmyeon, yongsanmyeon, yongdongeup } from './geojson';
import doughnut from '@/components/chart/doughnut'
import dashboardBarchartDay from '@/components/chart/dashboardBarchartDay'
import dashboardBarchartMonth from '@/components/chart/dashboardBarchartMonth'
import supply from '@/components/item/supply'

export default {
  name: 'ydn',
  components: { doughnut, dashboardBarchartDay, dashboardBarchartMonth, supply },
  data(){
    return{
    dashboard: true,
    home: false,
    currDay: '',
    map: null,
    simcheonmyeon:simcheonmyeon,
    yangsanmyeon: yangsanmyeon,
    haksanmyeon: haksanmyeon,
    yonghawmyeon: yonghawmyeon,
    yanggangmyeon: yanggangmyeon,
    sangchonmyeon: sangchonmyeon,
    maegokmyeon: maegokmyeon,
    chupungnyeongmyeon: chupungnyeongmyeon,
    hwangganmyeon: hwangganmyeon,
    yongsanmyeon: yongsanmyeon,
    yongdongeup:yongdongeup,
    customWindow:{
      width: '100%',
    //   height: (window.innerHeight - 54) + 'px',             //기존 화면 높이
      height: (window.innerHeight) + 'px',             //기존 화면 높이
    },
    markers: [],
    years: [],
    year: new Date().getFullYear(),
    months: ["전체", 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12],
    month: new Date().getMonth() + 1,
    selectAllList: [], //전체 지역 리스트
    selectData: "", //현재 선택한 지역의 데이터
    totalStatusList: [], //전체 마커들의 데이터
    resultData: { //총 합계의 데이터
      siteCnt: 0, //발전소 수
      equipSize: 0, //설비용량
      energyindi: 0, //에너지 자립율
      totalCurGen: 0, //발전량
      totalCurUsg: 0, //사용량
      carbonReduction: 0, //탄소저감량
      plantingEffect: 0 //식수효과
    },
    powerPlantList: [], //발전소운영현황 데이터
    defaultBjdongCd: "4374000000", //영덕군
    genByRegionList: [], //지역별 태양광 발전량
    genList: [], //발전량
    useList: [], //사용량
  }},
  created(){
    // 헤더 영역 셋팅
    let params = {
        title : '충청북도 영동군 태양광 통합관제 플랫폼',
        menuType : 1,
        isHeader : true,
        isFooter : true,
        isBtnBack : false,
        isBtnClose : false,
        isBtnRefresh : false,
        isBtnAlarm : false,
        gray: true,
        isFooter: false,
        isLoginButton: true,
        isBtnFullScreen: true,
    }
    this.$nuxt.$emit('SET_TITLE', params)

    // 공통 팝업 종료 후 해당 이벤트 버스를 통해 backkey를 다시 설정
    this.$nuxt.$off('SET_BACK_KEY_' + this.pageId)
    this.$nuxt.$on('SET_BACK_KEY_' + this.pageId, () => {
      this.fnSetBackKey()
    })

    // 앱 종료 팝업 취소 시 이벤트 버스를 통해 backkey를 다시 설정
    this.$nuxt.$off('SET_BACK_KEY_DEFAULT')
    this.$nuxt.$on('SET_BACK_KEY_DEFAULT', () => {
      this.fnSetBackKey()
    })

    // 화면 rewakeup 일경우 다시 조회
    this.$nuxt.$off('RESET_DASHBOARD')
    this.$nuxt.$on('RESET_DASHBOARD', () => {
      this.$log.debug("[index][RESET_DASHBOARD]");
      if(!this.$utils.isNull(this.search.ppt.pptSeq)) {
        this.$log.debug("[index][RESET_DASHBOARD][ACTION]");
        // 자동로그인 시 넘어왔을 경우에는 데시보드를 조회할 필요가 없다.
        if(this.isAutoLogin) this.isAutoLogin = false
        else this.fnGetDashboardDatas()
      }
    })

    // 네이티브 디스플레이 모드 조회
    this.$nuxt.$off('SET_DISPLAY_MODE')
    this.$nuxt.$on('SET_DISPLAY_MODE', (param) => {
      if(!this.$utils.isNull(param.visibility)) this.sysMode = param.visibility
    })
  },

  methods:{
    async fnGetDashboardDatas() {
      // 대시보드 데이터 초기화
      this.dbData = _.cloneDeep(this.initDbData)
      await this.fnSetToday()        // 새로고침 일자 변경
    },

    fnSetBackKey() {
      // 앱 종료 처리
      let params = {
        options: {
          action: 'js'
        },
        callback: {
          name: 'window.callbackObj.fnSetExitConfirm',
          param: {}
        }
      }
      this.$connect.ifBackKey(params)
    },

    //마커 클릭 이벤트
    selectRegion(index){
      let map = this.map;
      let detailHeader = $('.detail-header');
      let progress = $('.operation-progress');
      let energyindi = $('.marker-detailInfo-energyindi');
      
      return (e) =>{
        for(let i=0; i<map.data._features.length; i++){
          let data = map.data._features[i];
          if(i !== index){
            data.setProperty('focus', false);
            map.data.revertStyle(data);
            progress.eq(i).removeClass("focus");
            detailHeader[i].style.background = "#757CA5";
            energyindi[i].style.color = "black";
          }else{
            if (data.getProperty('focus') !== true) {
                detailHeader[i].style.background = "#324DFF";
                energyindi[i].style.color = "#324DFF";
                progress.eq(i).addClass("focus");
                data.setProperty('focus', true);
                this.selectData = data.property_EMD_CD + "00";
            } else {
                detailHeader[i].style.background = "#757CA5";
                energyindi[i].style.color = "black";
                progress.eq(i).removeClass("focus");
                data.setProperty('focus', false);
                this.selectData = "";
            }
          }
        }
      }
    },

    //마커 오버 이벤트
    hoverRegion(index){
      let markers = this.markers;
      return function(e){
        markers[index].setZIndex(1000); //마커 겹쳐있을때 zindex 설정
      }
    },

    //마커 오버아웃 이벤트
    hoveroutRegion(index){
      let markers = this.markers;
      return function(e){
        markers[index].setZIndex(100); //마커 겹쳐있을때 zindex 설정
      }
    },

    //geojson 영역
    setGeoLayout(map){
        map.data.setStyle(function(feature) {
            var mantle_properties = feature.geometryCollection[0].getRaw().mantle_properties;
            var styleOptions = {
                ...mantle_properties,
            };

            if (feature.getProperty('focus')) {
                styleOptions.fillOpacity = 1;
                styleOptions.fillColor = '#324DFF';
                styleOptions.strokeColor = 'white';
                styleOptions.strokeWeight = 1;
                styleOptions.strokeOpacity = 1;
            }
            return styleOptions;
        });
        
        map.data.addGeoJson(this.simcheonmyeon, true);
        map.data.addGeoJson(this.yangsanmyeon, true);
        map.data.addGeoJson(this.haksanmyeon, true);
        map.data.addGeoJson(this.yonghawmyeon, true);
        map.data.addGeoJson(this.yanggangmyeon, true);
        map.data.addGeoJson(this.sangchonmyeon, true);
        map.data.addGeoJson(this.maegokmyeon, true);
        map.data.addGeoJson(this.chupungnyeongmyeon, true);
        map.data.addGeoJson(this.hwangganmyeon, true);
        map.data.addGeoJson(this.yongsanmyeon, true);
        map.data.addGeoJson(this.yongdongeup, true);

        map.data.addListener('click', (e) =>{
            let detailHeader = $('.detail-header');
            let progress = $('.operation-progress');
            let energyindi = $('.marker-detailInfo-energyindi');

            var feature = e.feature;
            for(let i=0; i<map.data._features.length; i++){
              let data = map.data._features[i];
              
              if(feature.id !== data.id){
                data.setProperty('focus', false);
                map.data.revertStyle(data);
                detailHeader[i].style.background = "#757CA5";
                energyindi[i].style.color = "black";
                progress.eq(i).removeClass("focus");
              }else{
                if (feature.getProperty('focus') !== true) {
                    feature.setProperty('focus', true);
                    detailHeader[i].style.background = "#324DFF";
                    energyindi[i].style.color = "#324DFF";
                    progress.eq(i).addClass("focus");
                    this.selectData = data.property_EMD_CD + "00";
                } else {
                    feature.setProperty('focus', false);
                    detailHeader[i].style.background = "#757CA5";
                    energyindi[i].style.color = "black";
                    progress.eq(i).removeClass("focus");
                    this.selectData = "";
                }
              }
            }
        });

        map.data.addListener('mouseover', (e) =>{
            var feature = e.feature;
            map.data.overrideStyle(feature, {
                fillOpacity: 1,
                strokeWeight: 3,
                strokeOpacity: 1
            });
        });

        map.data.addListener('mouseout', (e) =>{
            map.data.revertStyle();
        });
    },

    //지역라벨
    setLable(map){
      for(let i=0; i<map.data._features.length; i++){
        let feature = map.data._features[i]
        let marker = new naver.maps.Marker({
          position: new naver.maps.LatLng((feature.bounds._max.y + feature.bounds._min.y) / 2, (feature.bounds._max.x + feature.bounds._min.x) / 2),
          map: map,
          icon: {
            content: [
                '<div class="markerText">',
                  `<span style="font-size:15px; color:white; font-weight:200; text-shadow: 2px 2px 2px gray;">` + feature.property_EMD_NM + `</span>`,
                '</div>'
            ].join(''),
            anchor: new naver.maps.Point(25, 10),
          },
        })
        marker.setMap(map)
      }

      let mark = $('.markerText');
      for(let i=0; i<mark.length; i++){
        let parents = mark[i].parentNode;
        parents.style.pointerEvents = "none";
      }
    },

    //상세정보
    setDetail(map){
      for(let i=0; i<map.data._features.length; i++){
        let feature = map.data._features[i]
        let x = feature.bounds._min.x;
        let y = feature.bounds._max.y;
        if(feature.property_EMD_NM === "상촌면"){
          x = (feature.bounds._min.x + feature.bounds._max.x) / 2;
          y = feature.bounds._min.y
        }else if(feature.property_EMD_NM === "추풍량면"){
          x = (feature.bounds._min.x + feature.bounds._max.x) / 2;
          y = feature.bounds._max.y + 0.02
        }else if(feature.property_EMD_NM === "매곡면"){
          x = feature.bounds._max.x;
          y = (feature.bounds._min.y + feature.bounds._max.y) / 2;
        }else if(feature.property_EMD_NM === "황간면"){
          x = feature.bounds._min.x + 0.02
          y = feature.bounds._max.y
        }else if(feature.property_EMD_NM === "용화면"){
          x = (feature.bounds._min.x + feature.bounds._max.x) / 2;
          y = feature.bounds._min.y
        }else if(feature.property_EMD_NM === "영동읍"){
          x = (feature.bounds._min.x + feature.bounds._max.x) / 2;
          y = (feature.bounds._min.y + feature.bounds._max.y) / 2;
        }else if(feature.property_EMD_NM === "양산면"){
          x = feature.bounds._min.x - 0.08;
          y = feature.bounds._max.y - 0.01;
        }else if(feature.property_EMD_NM === "심천면"){
          x = feature.bounds._min.x - 0.08;
          y = feature.bounds._max.y + 0.02;
        }else if(feature.property_EMD_NM === "양강면"){
          y = feature.bounds._max.y + 0.05;
        }else if(feature.property_EMD_NM === "학산면"){
          x = ((feature.bounds._min.x + feature.bounds._max.x) / 2) - 0.05;
          y = ((feature.bounds._min.y + feature.bounds._max.y) / 2) + 0.02;
        }else if(feature.property_EMD_NM === "용산면"){
          y = feature.bounds._max.y + 0.02;
        }
        
        
        let marker = new naver.maps.Marker({
          position: new naver.maps.LatLng(y, x),
          map: map,
          icon: {
            content: [
                `<div class="marker-detailInfo">`,
                  `<table>`,
                    `<thead>`,
                      `<tr>`,
                        `<th colspan="2" class="detail-header" style="border-top-left-radius: 10px; border-top-right-radius: 10px">`,
                          `<i class="ydn-logo"></i>`,
                          feature.property_EMD_NM,
                        `</th>`,
                      `</tr>`,
                    `</thead>`,
                    `<tbody class="marker-body">`,
                      `<tr>`,
                        `<td>`,
                          "RE달성률",
                        `</td>`,
                        `<td class="marker-detailInfo-energyindi" target=${feature.property_EMD_CD + "00"}>`,
                          0,
                          `<span> %</span>`,
                        `</td>`,
                      `</tr>`,
                      `<tr>`,
                        `<td colspan="2" style="text-align: center; padding: 2px 5px;">`,
                          `<progress class="operation-progress" target=${feature.property_EMD_CD + "00"} value="0" min="0" max="100"></progress>`,
                        `</td>`,
                      `</tr>`,
                      `<tr>`,
                        `<td>`,
                          "발전량",
                        `</td>`,
                        `<td class="marker-detailInfo-totalCurGen" target=${feature.property_EMD_CD + "00"}>`,
                          0,
                          `<span> MWh</span>`,
                        `</td>`,
                      `</tr>`,
                      `<tr>`,
                        `<td>`,
                          "소비량",
                        `</td>`,
                        `<td class="marker-detailInfo-totalCurUsg" target=${feature.property_EMD_CD + "00"}>`,
                          0,
                          `<span> MWh</span>`,
                        `</td>`,
                      `</tr>`,
                      `<tr>`,
                        `<td colspan="2" style="text-align: center; padding: 2px 5px; color: #C0C4D9">`,
                          "---------------------",
                        `</td>`,
                      `</tr>`,
                      `<tr>`,
                        `<td>`,
                          "발전소 수",
                        `</td>`,
                        `<td class="marker-detailInfo-siteCnt" target=${feature.property_EMD_CD + "00"}>`,
                          0,
                          `<span> 개</span>`,
                        `</td>`,
                      `</tr>`,
                      `<tr>`,
                        `<td style="border-bottom-left-radius: 10px">`,
                          "설비용량",
                        `</td>`,
                        `<td class="marker-detailInfo-equipSize" target=${feature.property_EMD_CD + "00"} style="border-bottom-right-radius: 10px" >`,
                          0,
                          `<span> MW</span>`,
                        `</td>`,
                      `</tr>`,
                    `</tbody>`,
                  `</table>`,
                '</div>'
            ].join(''),
            anchor: new naver.maps.Point(25, 10),
          },
        })
        this.markers.push(marker);
        marker.setMap(map)
      }

      //마커 클릭 이벤트 리스너 추가
      for (let i=0; i<this.markers.length; i++) {
        naver.maps.Event.addListener(this.markers[i], 'click', this.selectRegion(i));
      }

      //마커 오버 이벤트 리스너 추가
      for (let i=0; i<this.markers.length; i++) {
        naver.maps.Event.addListener(this.markers[i], 'mouseover', this.hoverRegion(i));
      }

      //마커 오버아웃 이벤트 리스너 추가
       for (let i=0; i<this.markers.length; i++) {
        naver.maps.Event.addListener(this.markers[i], 'mouseout', this.hoveroutRegion(i));
      }

      let mark = $('.markerText');
      for(let i=0; i<mark.length; i++){
        let parents = mark[i].parentNode;
        parents.style.pointerEvents = "none";
      }
    },

    initMap(){
        let mapOptions={
            center: new naver.maps.LatLng(36.107205441308, 127.755909717)
            , zoom: 11
            , scaleControl: false           //우하단 축척
            , logoControl: false            //네이버 로고 뭔지 모르겠음.
            , mapDataControl: false         //좌하단 네이버 로고
            , mapTypeControl: true          //지도 유형 컨트롤
            , zoomControl: false             //줌 컨트롤
            , disableDoubleClickZoom : true//더블 클릭해 지도를 확대하는 기능의 사용 여부
            , mapTypeId: naver.maps.MapTypeId.NORMAL//SATELLITE //HYBRID
            , scrollWheel: false //마우스 휠
            , draggable: false //마우스 드레그
            , mapTypeControl: false //지도 타입 설정 컨트롤
        }
        this.map = new naver.maps.Map('map',mapOptions)
        this.setGeoLayout(this.map);
        this.setLable(this.map);
        this.setDetail(this.map);
    },

    getToDay(){
        const day = ['일요일', '월요일', '화요일', '수요일', '목요일', '금요일', '토요일'];
        let today = new Date();
        let year = today.getFullYear();
        let month = ("00"+(today.getMonth()+1).toString()).slice(-2);
        let date = ("00"+today.getDate().toString()).slice(-2);
        let hour = ("00"+today.getHours().toString()).slice(-2);
        let minute = ("00"+today.getMinutes().toString()).slice(-2);
        let second = ("00"+today.getSeconds().toString()).slice(-2);

        let dateFormat = year + "-" + month + "-" + date + 
        " " + day[today.getDay()] + " " + hour + ":" + minute + ":" + second;
        this.currDay = dateFormat;
    },

    setYears(){
      let tempList = [];
      for(let i=2022; i<this.year + 1; i++){
        tempList.push(i);
      }
      tempList = [...new Set(tempList.sort())]
      this.years = tempList;
    },

    setSelectAllList(){
      let tempBjdongList = [];
      tempBjdongList.push(simcheonmyeon.features[0].properties.EMD_CD + "00");
      tempBjdongList.push(yangsanmyeon.features[0].properties.EMD_CD + "00");
      tempBjdongList.push(haksanmyeon.features[0].properties.EMD_CD + "00");
      tempBjdongList.push(yonghawmyeon.features[0].properties.EMD_CD + "00");
      tempBjdongList.push(yanggangmyeon.features[0].properties.EMD_CD + "00");
      tempBjdongList.push(sangchonmyeon.features[0].properties.EMD_CD + "00");
      tempBjdongList.push(maegokmyeon.features[0].properties.EMD_CD + "00");
      tempBjdongList.push(chupungnyeongmyeon.features[0].properties.EMD_CD + "00");
      tempBjdongList.push(hwangganmyeon.features[0].properties.EMD_CD + "00");
      tempBjdongList.push(yongsanmyeon.features[0].properties.EMD_CD + "00");
      tempBjdongList.push(yongdongeup.features[0].properties.EMD_CD + "00");

      this.selectAllList = tempBjdongList;
    },

    selectTotalStatus(){
      let monthData = "";
      if(this.month === "전체"){
        monthData = "";
      }else{
        monthData = this.month;
      }

      let params = {
        bjdongCdList: this.selectAllList,
        periodDateYear: this.year,
        periodDateMonth: monthData
      }

      this.$api.post('/rm/site/region/energy-stats', params)
      .then((res)=>{
        this.totalStatusList = res.data.bjdongList;
        this.totalData();
      }).catch((error)=>{
        this.totalStatusList = [];
      })
    },

    //테스트
    selectEnergyStats(){
      let stDt = "";
      let endDt = "";
      let tempBjdongCd = "";

      if(this.month === "전체"){
        stDt = this.year.toString() + "01";
        endDt = this.year.toString() + "12";
      }else{
        if(this.month < 10){
          stDt = this.year.toString() + "0" + this.month.toString();
          endDt = this.year.toString() + "0" + this.month.toString();
        }else{
          stDt = this.year.toString() + this.month.toString();
          endDt = this.year.toString() + this.month.toString();
        }
      }

      tempBjdongCd = this.defaultBjdongCd.substr(0, 5)
      let params = {
        bjdongCd: tempBjdongCd,
        stDt: stDt,
        endDt: endDt
      }

      this.$api.post('/rm/statistics/selectEnergyStats', params)
      .then((res)=>{
        this.totalStatusList = res.data.bjdongList;
        this.totalData();
      }).catch((error)=>{
        this.totalStatusList = [];
      })
    },

    //발전소운영현황
    selectPowerPlantStatus(){
      let tempBjdongCd = "";
      tempBjdongCd = this.defaultBjdongCd.substr(0, 5);

      let params = {
        bjdongCd: tempBjdongCd,
      }

      this.$api.post('/rm/site/region/powerPlant-stats', params)
      .then((res)=>{
        this.powerPlantList = res.data.powerPlantList;
      }).catch((error)=>{
        this.powerPlantList = [];
      })
    },

    totalData(){
      let res = {
        siteCnt: 0, //발전소 수
        equipSize: 0, //설비용량
        energyindi: 0, //에너지 자립율
        totalCurGen: 0, //발전량
        totalCurUsg: 0, //사용량
        carbonReduction: 0, //탄소저감량
        plantingEffect: 0 //식수효과
      }

      let ei = 0;
      let gen = 0;
      let usg = 0;
      let cnt = 0;
      let vol = 0;
      let cr = 0;
      let pe = 0;

      if(this.selectData != ""){
        for(let i=0; i<this.totalStatusList.length; i++){
          if(this.totalStatusList[i].bjdongCd === this.selectData){
            gen = this.totalStatusList[i].curGen;
            usg = this.totalStatusList[i].curUsg; //소비량
            cnt = this.totalStatusList[i].equipCnt; //발전소 수
            vol = this.totalStatusList[i].capa; //설비용량
            ei = this.totalStatusList[i].re; //re달성률(에너지 자립률)
            cr = gen * 0.424;
            pe = (cr * 1000) / 2.35;

            res.siteCnt = cnt;
            res.equipSize = vol;
            res.energyindi = ei;
            res.totalCurGen = gen;
            res.totalCurUsg = usg;
            res.carbonReduction = cr;
            res.plantingEffect = pe;
          }
        }
      }else{
        for(let i=0; i<this.totalStatusList.length; i++){
          gen = this.totalStatusList[i].curGen;
          usg = this.totalStatusList[i].curUsg; //소비량
          cnt = this.totalStatusList[i].equipCnt; //발전소 수
          vol = this.totalStatusList[i].capa; //설비용량
          // ei = this.totalStatusList[i].re; //re달성률(에너지 자립률)
          cr = gen * 0.424;
          pe = (cr * 1000) / 2.35;

          res.siteCnt = res.siteCnt + parseInt(cnt);
          res.equipSize = res.equipSize + parseInt(vol);
          // res.energyindi = res.energyindi + parseInt(ei);
          res.totalCurGen = res.totalCurGen + parseInt(gen);
          res.totalCurUsg = res.totalCurUsg + parseInt(usg);
          res.carbonReduction = res.carbonReduction + parseInt(cr);
          res.plantingEffect = res.plantingEffect + parseInt(pe);
        }
        res.energyindi = res.totalCurGen / res.totalCurUsg * 100; //전체 re달성률(에너지 자립률)
      }
      this.resultData = res
    },

    //w값 => kw값 or kw => mw 변경
    fn_roundWToKw(value){
      return Math.round(value / 10000) / 100;    // / 1000   /1000
    },

    //w 값 => mw값 변경
    fn_roundWToMw(value){
      return Math.round(value / 100000000) / 100;
    },

    //지역별 태양광 발전량
    selectGenByRegion(){
      let tempBjdongCd = "";

      let monthData = 0;
      if(this.month === "전체"){
        monthData = 0;
      }else{
        monthData = this.month;
      }

      tempBjdongCd = this.defaultBjdongCd.substr(0, 5)
      let params = {
        periodDateYear: this.year,
        periodDateMonth: monthData,
        bjdongCd: tempBjdongCd,
      }

      this.$api.post('/rm/statistics/selectGenStats', params)
      .then((res)=>{
        this.genByRegionList = res.data;
      }).catch((error)=>{
        this.genByRegionList = [];
      })
    },

    //태양광 발전량 추이
    selectSolarPower(){
      let tempBjdongCd = "";
      tempBjdongCd = this.defaultBjdongCd.substr(0, 5)

      let params = {
        periodDateYear: this.year,
        bjdongCd: tempBjdongCd,
      }

      this.$api.post('/rm/statistics/selectGenByMonthStats', params)
      .then((res)=>{
        this.genList = res.data.genList;
      }).catch((error)=>{
        this.genList = [];
      })
    },
    
    //사용량 추이
    selectUseTrand(){
      let tempBjdongCd = "";
      tempBjdongCd = this.defaultBjdongCd.substr(0, 5)

      let params = {
        periodDateYear: this.year,
        bjdongCd: tempBjdongCd,
      }

      this.$api.post('/rm/statistics/selectUseByMonthStats', params)
      .then((res)=>{
        this.useList = res.data.useList;
      }).catch((error)=>{
        this.useList = [];
      })
    },
  },

  watch:{
    'currDay'(){
        this.$nuxt.$emit('chgToday', this.currDay);
    },
    'year'(){
    //   this.selectEnergyStats();
    //   this.selectGenByRegion();
    //   this.selectSolarPower();
    //   this.selectUseTrand();
    },
    'month'(){
    //   this.selectGenByRegion();
    //   this.selectEnergyStats();
    },
    selectAllList: {
      deep: true,
      handler(){
        // this.selectEnergyStats();
      }
    },
    'selectData'(){
      this.totalData();
    },
    resultData: {
      handler(){
        let resData = this.resultData
        $('.count-num').each(function(){
          let $this = $(this);
          let target = $this.attr('target');
          let countTo = isNaN(resData[$this.attr('target')]) ? 0 : resData[$this.attr('target')];
          let startCount = $this.attr('start-count');
          let unit = $this.attr('unit');
          let valueText = $this.contents()[0];

          $({ countNum: startCount}).animate({
            countNum: countTo
          },
          {
            duration: 1500, 
            easing:'linear',
            step: function() {
              if(target === "siteCnt" || target === "energyindi" || target === "plantingEffect"){
                valueText.textContent = (Math.ceil(this.countNum)).toLocaleString() + " ";
              }else{
                // $this.text(Math.floor(this.countNum) + " " + unit);
                valueText.textContent = (Math.ceil(this.countNum * 100)/100).toLocaleString() + " ";
              }
            },
            complete: function() { 
              // $this.text(this.countNum + " " + unit);
              valueText.textContent = (Math.ceil(this.countNum * 100)/100).toLocaleString() + " ";
            }
          });
        })
      }
    },
    totalStatusList: {
      deep: true,
      handler(){
        let energyindi = $('.marker-detailInfo-energyindi');
        let siteCnt = $('.marker-detailInfo-siteCnt');
        let equipSize = $('.marker-detailInfo-equipSize');
        let totalCurGen = $('.marker-detailInfo-totalCurGen');
        let totalCurUsg = $('.marker-detailInfo-totalCurUsg');
        let progress = $('.operation-progress');

        let ei = 0;
        let gen = 0;
        let usg = 0;
        let cnt = 0;
        let vol = 0;
        let rate = 0;

        if(this.totalStatusList.length > 0){
          for(let i=0; i<this.totalStatusList.length; i++){
            for(let j=0; j<this.map.data._features.length; j++){
              if(this.totalStatusList[i].bjdongCd === this.map.data._features[j].property_EMD_CD + "00"){
                gen = this.totalStatusList[i].curGen; //발전량
                usg = this.totalStatusList[i].curUsg; //소비량
                cnt = this.totalStatusList[i].equipCnt; //발전소 수
                vol = this.totalStatusList[i].capa; //설비용량
                ei = this.totalStatusList[i].re; //re달성률(에너지 자립률)
                rate = ei; //%

                for(let q=0; q<energyindi.length; q++){
                  if(energyindi.eq(q).attr("target") === this.totalStatusList[i].bjdongCd){
                    energyindi.eq(q).contents()[0].textContent = ei;
                    siteCnt.eq(q).contents()[0].textContent = cnt;
                    equipSize.eq(q).contents()[0].textContent = vol;
                    totalCurGen.eq(q).contents()[0].textContent = gen;
                    totalCurUsg.eq(q).contents()[0].textContent = usg;
                    progress.eq(q)[0].value = rate;
                  }
                }
              }
            }
          }
        }else{
          for(let q=0; q<energyindi.length; q++){
            energyindi.eq(q).contents()[0].textContent = 0;
            siteCnt.eq(q).contents()[0].textContent = 0;
            equipSize.eq(q).contents()[0].textContent = 0;
            totalCurGen.eq(q).contents()[0].textContent = 0;
            totalCurUsg.eq(q).contents()[0].textContent = 0;
            progress.eq(q)[0].value = 0;
          }
        }
      },
    }
  },

  mounted(){
    this.$nuxt.$emit('chgGray', this.gray);
    this.$nuxt.$emit('chgDashbord', this.dashboard);
    
    this.initMap()
    setInterval(this.getToDay, 1000);

    window.addEventListener('resize', ()=>{
      this.customWindow.width = (window.innerWidth) + 'px';
      this.customWindow.height = (window.innerHeight) + 'px';
      this.map.setSize(this.customWindow);
    })

    //년도 셋팅
    this.setYears();

    //bjdong 리스트 전체 선택
    this.setSelectAllList();

    //총 현황
    // this.selectTotalStatus();

    //발전소 운영현황
    // this.selectPowerPlantStatus();

    //지역별 발전량
    // this.selectGenByRegion();

    //발전량 추이
    // this.selectSolarPower();

    //사용량 추이
    // this.selectUseTrand();

    //테스트
    // this.selectEnergyStats();
  },

  beforeDestroy(){
    window.removeEventListener('resize', ()=>{
    })
  },
}
</script> 

<style scoped>
</style>