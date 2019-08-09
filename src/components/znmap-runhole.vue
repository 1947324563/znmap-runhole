<template>
    <!-- -->
    <div class="container">
        <v-chart ref="instance"
                 :options="opt"
                @click="initChartOpt"
                 autoresize></v-chart>

    </div>
</template>
<script>
    import {cityMap} from '@/static/map/china-main-city-map.js';
    import chinaMapjson from '@/static/map/100000.json'
    import vEcharts from 'vue-echarts';
    import axios from 'axios';
    //中国地图（第一级地图）的ID、Name、Json数据
    // var chinaId = 100000;
    // var chinaName = 'china';
    // var chinaJson = null;
    //
    // //记录父级ID、Name
    // var mapStack = [];
    // var parentId = null;
    // var parentName = null;

    //Echarts地图全局变量，主要是在返回上级地图的方法中会用到
    // var myChart = null;
    export default {
        name: 'znmap-runhole',
        components: {
            'v-chart': vEcharts
        },
        props: {
            options: {
                type: Object,
                // required: true
            },
            isSetOptions: {
                type: Boolean,
                // default: true
            },
            isShowLoading: {
                type: Boolean,
                // default: false
            },
            h: {
                type: Number,
                // required: true
            },
            w: {
                type: Number,
                // required: true
            }
        },

        data() {
            return {
                chinaId:100000,
                chinaName : 'china',
                chinaJson:null,
                mapStack: [],
                parentId: null,
                parentName: null,
                opt:{series: [
                        {
                            type: 'map',
                            map: name,
                            itemStyle: {
                                normal: {
                                    areaColor: 'rgba(23, 27, 57,0)',
                                    borderColor: '#1dc199',
                                    borderWidth: 1
                                }
                            },
                            data: this.initMapData(chinaMapjson)
                        }
                    ]},
                myChart:this.$refs.instance,
                getHost:window.location.host,
            };
        },
        mounted(){
            this.initChart();
        },
        methods: {
            initChartOpt(params){
                console.log(params)
                let cityId = cityMap[params.name];
                if (cityId) {
                    //代表有下级地图
                    axios
                        .get(this.getHost + '/mapjson/map/' + cityId + '.json', {})
                        .then(response => {
                            // console.log(response);

                            this.mapJson = response.data;
                            this.registerAndsetOption(
                                this.myChart,
                                cityId,
                                params.name,
                                this.mapJson,
                                true
                            );
                        });
                } else {
                    //没有下级地图，回到一级中国地图，并将mapStack清空
                    this.registerAndsetOption(this.myChart, this.chinaId, this.chinaName, this.chinaJson, false);
                    this.mapStack = [];
                    this.parentId = this.chinaId;
                    this.parentName = this.chinaName;
                }
            },
            initChart(id){
                // console.log(chinaMapjson,'address')
                // axios.get('http://192.168.199.249' + '/static/map/' + this.chinaId + '.json', {}).then(response => {
                    this.mapJson = chinaMapjson;
                    this.chinaJson = this.mapJson;
                    this.registerAndsetOption(this.myChart, this.chinaId, this.chinaName, this.mapJson, false);
                    this.parentId = this.chinaId;
                    this.parentName = 'china';
                    // myChart.on('click', function(param) {
                    //   // console.log(cityMap);
                    // 	var cityId = cityMap[param.name];
                    // 	if (cityId) {
                    // 		//代表有下级地图
                    // 		axios
                    // 			.get('http://bivue.24e.test/static/map/' + cityId + '.json', {})
                    // 			.then(response => {
                    // 				// console.log(response);
                    // 				const mapJson = response.data;
                    // 				registerAndsetOption(
                    // 					myChart,
                    // 					cityId,
                    // 					param.name,
                    // 					mapJson,
                    // 					true
                    // 				);
                    // 			});
                    // 	} else {
                    // 		//没有下级地图，回到一级中国地图，并将mapStack清空
                    // 		registerAndsetOption(myChart, chinaId, chinaName, chinaJson, false);
                    // 		mapStack = [];
                    // 		parentId = chinaId;
                    // 		parentName = chinaName;
                    // 	}
                    // 	// $.get('./asset/json/map/'+param.data.id+'.json', function (mapJson,status) {
                    //
                    // 	//     registerAndsetOption(myChart,param.data.id,param.name,mapJson,true)
                    //
                    // // }).fail(function () {
                    // //     registerAndsetOption(myChart,chinaId,'china',chinaJson,false)
                    // // });
                    // });
                // });
            },
            registerAndsetOption(myChart, id, name, mapJson, flag){
                console.log(name)
                vEcharts.registerMap(name, mapJson);
                this.opt = {series: [
                        {
                            type: 'map',
                            map: name,
                            itemStyle: {
                                normal: {
                                    areaColor: 'rgba(23, 27, 57,0)',
                                    borderColor: '#1dc199',
                                    borderWidth: 1
                                }
                            },
                            data: this.initMapData(mapJson)
                        }
                    ],};
                console.log(this.opt)
                if (flag) {
                    //往mapStack里添加parentId，parentName,返回上一级使用
                    this.mapStack.push({
                        mapId: this.parentId,
                        mapName: this.parentName
                    });
                    this.parentId = id;
                    this.parentName = name;
                }
            },
            initMapData(mapJson){
                let mapData = [];
                for (let i = 0; i < mapJson.features.length; i++) {
                    mapData.push({
                        name: mapJson.features[i].properties.name
                        //id:mapJson.features[i].id
                    });
                }
                return mapData;
            }
        },
    };
    // function registerAndsetOption(myChart, id, name, mapJson, flag) {
    // 	vEcharts.registerMap(name, mapJson);
    // 	alert(11)
    // 	this.opt = {series: [
    // 		  {
    // 			  type: 'map',
    // 			  map: name,
    // 			  itemStyle: {
    // 				  normal: {
    // 					  areaColor: 'rgba(23, 27, 57,0)',
    // 					  borderColor: '#1dc199',
    // 					  borderWidth: 1
    // 				  }
    // 			  },
    // 			  data: initMapData(mapJson)
    // 		  }
    // 	  ],};
    // 	// myChart.setOption({
    // 	// 	series: [
    // 	// 		{
    // 	// 			type: 'map',
    // 	// 			map: name,
    // 	// 			itemStyle: {
    // 	// 				normal: {
    // 	// 					areaColor: 'rgba(23, 27, 57,0)',
    // 	// 					borderColor: '#1dc199',
    // 	// 					borderWidth: 1
    // 	// 				}
    // 	// 			},
    // 	// 			data: initMapData(mapJson)
    // 	// 		}
    // 	// 	],
    // 	// 	title: {
    // 	// 		text: '全国行政区划地图',
    // 	// 		x: 'center',
    // 	// 		top: '20',
    // 	// 		textStyle: {
    // 	// 			color: '#000',
    // 	// 			fontSize: 24
    // 	// 		}
    // 	// 	},
    // 	// 	tooltip: {
    // 	// 		show: true,
    // 	// 		// formatter:(params)=>{
    // 	// 		//   let data = "测试1:"+params.name + "<br/>"+"值:"+ params.value[2]+"<br/>"+"地理坐标:[" + params.value[0]+","+params.value[1] +"]";
    // 	// 		//   return data;
    // 	// 		// },
    // 	// 	},
    // 	// 	grid: {
    // 	// 		// 顯隱網格綫
    // 	// 		show: true,
    // 	// 		// 网格线颜色
    // 	// 		borderColor: 'rgba(44,111,198,0.5)',
    // 	// 		// 網格寬度
    // 	// 		borderWidth: 0,
    // 	// 		width: '80%',
    // 	// 		height: '80%',
    // 	// 		top: '10%',
    // 	// 		left: '10%',
    // 	// 		// right: '10%',
    // 	// 		// bottom:'10%',
    // 	// 		containLabel: true,
    // 	// 	},
    // 	// });
    //
    // 	if (flag) {
    // 		//往mapStack里添加parentId，parentName,返回上一级使用
    // 		mapStack.push({
    // 			mapId: parentId,
    // 			mapName: parentName
    // 		});
    // 		parentId = id;
    // 		parentName = name;
    // 	}
    // }
    function initMapData(mapJson) {
        var mapData = [];
        for (var i = 0; i < mapJson.features.length; i++) {
            mapData.push({
                name: mapJson.features[i].properties.name
                //id:mapJson.features[i].id
            });
        }
        return mapData;
    }
</script>
<style scoped>

</style>
