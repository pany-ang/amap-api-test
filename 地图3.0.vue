<template>
  <div>
    <navigationBarNew @updateButtonState="updateButtonState"></navigationBarNew>
    <el-autocomplete v-if="searchFlag" v-model="state" :fetch-suggestions="querySearchAsync" placeholder="搜索小区" @select="handleSelect" class="search"></el-autocomplete>
    <div id="container"></div>
  </div>
</template>

<script>
let than;
import navigationBarNew from '@/components/NavigationBarNew/index'
export default {
  name: "Non_visitor_new_map",
  components: { navigationBarNew },
  computed: {
    searchFlag() {
      return than.nowMapState === 2 ? true : false;
    }
  },
  data() {
    return {
      isCollapse: true,
      map: null,
      center: [106.532428, 29.566606], //地图中心点（解放碑附近）
      buttonState: 2, // 当前用户选择了什么按钮：居民:0 商圈:1 板块:2
      nowMapState: 0, // 当前地图处于那一层：行政区:0 板块或商圈:1 小区:2
      district: null,
      polygons: [], // 多边形覆盖物
      markerList: [], //覆盖物
      marker: null, // 中间变量
      buttonStateString : '', // 根据buttonState来决定小区/商圈/板块
      count: 0, // 小区/商圈/板块的个数
      id0: 999, // 记录点击了哪一个行政区
      id1: 999, // 用户点击了哪一个小区板块
      restaurants: [], // 搜索框相关：原始数据
      state: "", // 搜索框相关：输入的内容
      timeout: null, // 搜索框相关：定时器开关
      cantonArr: [
        // 行政区
        {
          lnglat: [106.529378, 29.54256],
          name: "渝中",
          adcode: "500103"
        },
        {
          lnglat: [106.384708, 29.60121],
          name: "沙坪坝",
          adcode: "500106"
        },
        {
          lnglat: [106.643192, 29.501327],
          name: "南岸",
          adcode: "500108"
        },
        {
          lnglat: [106.409296, 29.482821],
          name: "九龙坡",
          adcode: "500107"
        },
        {
          lnglat: [106.573498, 29.607349],
          name: "江北",
          adcode: "500105"
        },
        {
          lnglat: [106.393725, 29.805215],
          name: "北碚",
          adcode: "500109"
        },
        {
          lnglat: [106.537921, 29.404028],
          name: "巴南",
          adcode: "500113"
        },
        {
          lnglat: [106.461616, 29.416545],
          name: "大渡口",
          adcode: "500104"
        },
        {
          lnglat: [106.627185, 29.716996],
          name: "渝北",
          adcode: "500112"
        }
      ],
      estateArr: [
        // 板块
        {
          estateArr: [
            {
              lnglat: [106.584385, 29.570546], // 代表点
              lnglatRange: [
                [106.584385, 29.570546],
                [106.595668, 29.56351],
                [106.595021, 29.576075],
                [106.584385, 29.570546]
              ], // 范围
              name: "朝天门"
            }
          ],
          name: "渝中"
        },
        {
          estateArr: [
            {
              lnglat: [106.303191, 29.612153],
              lnglatRange: [
                [106.313191, 29.642153],
                [106.322863, 29.645264],
                [106.341404, 29.642753],
                [106.339679, 29.63974],
                [106.336948, 29.630447],
                [106.337308, 29.605203],
                [106.334721, 29.585669],
                [106.285278, 29.585857],
                [106.284559, 29.592013],
                [106.297279, 29.622598]
              ],
              name: "大学城西"
            },
            {
              lnglat: [106.399677, 29.671605],
              lnglatRange: [
                [106.399677, 29.671605],
                [106.355625, 29.673927],
                [106.352606, 29.656478],
                [106.357206, 29.630613],
                [106.399103, 29.630864],
                [106.399677, 29.671605]
              ],
              name: "万达文旅城+土主+物流园"
            }
          ],
          name: "沙坪坝"
        },
        {
          estateArr: [
            {
              lnglat: [106.659213, 29.531786],
              lnglatRange: [
                [106.659213, 29.531786],
                [106.682784, 29.519466],
                [106.67531, 29.5079],
                [106.670423, 29.503373],
                [106.649439, 29.513683],
                [106.653176, 29.522232],
                [106.659213, 29.531786]
              ],
              name: "茶园南"
            }
          ],
          name: "南岸"
        },
        {
          estateArr: [
            {
              lnglat: [106.293604, 29.561709],
              lnglatRange: [
                [106.293604, 29.561709],
                [106.394214, 29.565228],
                [106.390765, 29.505899],
                [106.384441, 29.45358],
                [106.344772, 29.459618],
                [106.312002, 29.452071],
                [106.299353, 29.510928],
                [106.293604, 29.561709]
              ],
              name: "白市驿"
            }
          ],
          name: "九龙坡"
        },
        {
          estateArr: [
            {
              lnglat: [106.566994, 29.592659],
              lnglatRange: [
                [106.566994, 29.592659],
                [106.582732, 29.591465],
                [106.591212, 29.57463],
                [106.591212, 29.57463],
                [106.566994, 29.592659]
              ],
              name: "江北嘴"
            }
          ],
          name: "江北"
        },
        {
          estateArr: [
            {
              lnglat: [106.495252, 29.786428],
              lnglatRange: [
                [106.495252, 29.786428],
                [106.459032, 29.750813],
                [106.528022, 29.719201],
                [106.543545, 29.731245],
                [106.525148, 29.767117],
                [106.495252, 29.786428]
              ],
              name: "蔡家北"
            }
          ],
          name: "北碚"
        },
        {
          estateArr: [
            {
              lnglat: [106.546882, 29.392416],
              lnglatRange: [
                [106.546882, 29.392416],
                [106.600062, 29.384612],
                [106.601499, 29.376051],
                [106.591438, 29.350365],
                [106.538546, 29.341046],
                [106.541995, 29.369756],
                [106.546882, 29.392416]
              ],
              name: "龙洲湾南"
            }
          ],
          name: "巴南"
        },
        {
          estateArr: [
            {
              lnglat: [106.389068, 29.379325],
              lnglatRange: [
                [106.389068, 29.379325],
                [106.42155, 29.40601],
                [106.437361, 29.415575],
                [106.476167, 29.419099],
                [106.498877, 29.417337],
                [106.505201, 29.410289],
                [106.479904, 29.404248],
                [106.446847, 29.375296],
                [106.430174, 29.354647],
                [106.389643, 29.341298],
                [106.389068, 29.379325],
                [106.389068, 29.379325]
              ],
              name: "跳蹬"
            }
          ],
          name: "大渡口"
        },
        {
          estateArr: [
            {
              lnglat: [106.604759, 29.706316],
              lnglatRange: [
                [106.604759, 29.706316],
                [106.627037, 29.698661],
                [106.605621, 29.669919],
                [106.573713, 29.684103],
                [106.604759, 29.706316]
              ],
              name: "回兴"
            }
          ],
          name: "渝北"
        }
      ],
      commercialArr: [
        //商圈
        {
          commercialArr: [
            {
              lnglat: [106.307499, 29.613535], // 代表点
              lnglatRange: [
                [106.307499, 29.613535],
                [106.310625, 29.613567],
                [106.310517, 29.607004],
                [106.313643, 29.607067],
                [106.313463, 29.6049],
                [106.313463, 29.6049]
              ], // 范围
              name: "熙街"
            }
          ],
          name: "渝中"
        },
        {
          commercialArr: [
            {
              lnglat: [106.307499, 29.613535],
              lnglatRange: [
                [106.307499, 29.613535],
                [106.310625, 29.613567],
                [106.310517, 29.607004],
                [106.313643, 29.607067],
                [106.313463, 29.6049],
                [106.313463, 29.6049]
              ],
              name: "熙街"
            }
          ],
          name: "沙坪坝"
        },
        {
          commercialArr: [
            {
              lnglat: [106.307499, 29.613535],
              lnglatRange: [
                [106.307499, 29.613535],
                [106.310625, 29.613567],
                [106.310517, 29.607004],
                [106.313643, 29.607067],
                [106.313463, 29.6049],
                [106.313463, 29.6049]
              ],
              name: "熙街"
            }
          ],
          name: "南岸"
        },
        {
          commercialArr: [
            {
              lnglat: [106.307499, 29.613535],
              lnglatRange: [
                [106.307499, 29.613535],
                [106.310625, 29.613567],
                [106.310517, 29.607004],
                [106.313643, 29.607067],
                [106.313463, 29.6049],
                [106.313463, 29.6049]
              ],
              name: "熙街"
            }
          ],
          name: "九龙坡"
        },
        {
          commercialArr: [
            {
              lnglat: [106.54107, 29.585579],
              lnglatRange: [
                [106.54107, 29.585579],
                [106.541568, 29.584382],
                [106.541568, 29.584382],
                [106.538905, 29.583691]
              ],
              name: "北城天街"
            }
          ],
          name: "江北"
        },
        {
          commercialArr: [
            {
              lnglat: [106.307499, 29.613535],
              lnglatRange: [
                [106.307499, 29.613535],
                [106.310625, 29.613567],
                [106.310517, 29.607004],
                [106.313643, 29.607067],
                [106.313463, 29.6049],
                [106.313463, 29.6049]
              ],
              name: "熙街"
            }
          ],
          name: "北碚"
        },
        {
          commercialArr: [
            {
              lnglat: [106.307499, 29.613535],
              lnglatRange: [
                [106.307499, 29.613535],
                [106.310625, 29.613567],
                [106.310517, 29.607004],
                [106.313643, 29.607067],
                [106.313463, 29.6049],
                [106.313463, 29.6049]
              ],
              name: "熙街"
            }
          ],
          name: "巴南"
        },
        {
          commercialArr: [
            {
              lnglat: [106.307499, 29.613535],
              lnglatRange: [
                [106.307499, 29.613535],
                [106.310625, 29.613567],
                [106.310517, 29.607004],
                [106.313643, 29.607067],
                [106.313463, 29.6049],
                [106.313463, 29.6049]
              ],
              name: "熙街"
            }
          ],
          name: "大渡口"
        },
        {
          commercialArr: [
            {
              lnglat: [106.307499, 29.613535],
              lnglatRange: [
                [106.307499, 29.613535],
                [106.310625, 29.613567],
                [106.310517, 29.607004],
                [106.313643, 29.607067],
                [106.313463, 29.6049],
                [106.313463, 29.6049]
              ],
              name: "熙街"
            }
          ],
          name: "渝北"
        }
      ],
      houseArr: [
        {
          // 小区
          houseArr: [
            {
              lnglat: [106.309123, 29.601514],
              name: "富力城倾城里",
              value: "富力城倾城里"
            },
            {
              lnglat: [106.287236, 29.605096],
              name: "龙湖U城听蓝湾2期",
              value: "龙湖U城听蓝湾2期"
            },
            {
              lnglat: [106.31762, 29.612708],
              name: "龙湖·花千树2期",
              value: "龙湖·花千树2期"
            },
            {
              lnglat: [106.322826, 29.620488],
              name: "龙湖花千树",
              value: "龙湖花千树"
            },
            {
              lnglat: [106.316589, 29.61571],
              name: "富力城A区",
              value: "富力城A区"
            },{
              lnglat: [106.294195, 29.609045],
              name: "龙湖听蓝湾2期",
              value: "龙湖听蓝湾2期"
            }
          ],
          name: "大学城西"
        }
      ]
    };
  },
  methods: {
    // 搜索框
    querySearchAsync(queryString, cb) {
      var restaurants = this.restaurants;
      var results = queryString ? restaurants.filter(this.createStateFilter(queryString)) : restaurants;
      clearTimeout(this.timeout);
      this.timeout = setTimeout(() => {
        cb(results);
      }, 500 * Math.random()); // 防抖
    },
    createStateFilter(queryString) {
      return state => {
        return (
          state.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0
        );
      };
    },
    // 用户搜索了某一个小区时触发
    handleSelect(item) {
      for(let i=0; i<than.markerList.length; i++){
        if(than.markerList[i].B.extData.name === item.name){
          than.markerList[i].setzIndex(200); // 显示在最上层
          than.markerList[i].setContent(`<div style="position: relative;width: 200px;padding:3px 10px;border-radius: 10px;background-color: rgba(243, 49, 49, 0.9);color: white;text-align: center;">${than.markerList[i].B.extData.name}<div style="width:0px;height:0px;border-width:8px 8px 0px 8px;border-style:solid;border-color:rgba(243, 49, 49, 0.9) transparent transparent transparent;position:absolute;buttom:0;left:50%;"></div></div>`);
          than.map.setCenter(than.markerList[i].B.extData.lnglat);
          break;
        }
      }
    },
    //更新左侧按钮状态
    updateButtonState(val){
      localStorage.setItem('buttonState', val);
      than.buttonState = val;
      than.nowMapState = 0;
      than.map.remove(than.markerList);
      than.map.remove(than.polygons);
      than.addMarker(than.cantonArr);
      than.Event();
      than.map.setZoom(11);
      than.map.setCenter(than.center);
    },
    // 加载行政区
    drawBounds: function(value) {
      //加载行政区划插件
      if (!than.district) {
        //实例化DistrictSearch
        var opts = {
          subdistrict: 0, //获取边界不需要返回下级行政区
          extensions: "all", //返回行政区边界坐标组等具体信息
          level: "district" //查询行政级别为 市
        };
        than.district = new AMap.DistrictSearch(opts);
      }
      //行政区查询
      than.district.setLevel("district");
      than.district.search(value, function(status, result) {
        // console.log(result);
        than.map.remove(than.polygons); //清除上次结果
        than.polygons = [];
        var bounds = result.districtList[0].boundaries;
        if (bounds) {
          for (var i = 0, l = bounds.length; i < l; i++) {
            //生成行政区划polygon
            var polygon = new AMap.Polygon({
              strokeWeight: 1,
              path: bounds[i],
              fillOpacity: 0.4,
              fillColor: "#80d8ff",
              strokeColor: "#0091ea"
            });
            than.polygons.push(polygon);
          }
        }
        than.map.add(than.polygons);
      });
    },
    // 添加覆盖物
    addMarker: function(arr, val) {
      if(than.buttonState===2){
        than.buttonStateString = '板块'
      }else if(than.buttonState===1){
        than.buttonStateString = '商圈'
      }else if(than.buttonState===0){
        than.buttonStateString = '小区'
      }
      than.markerList = [];
      if (val === "小区") {
        for (let i = 0; i < arr.length; i++) {
          than.marker = new AMap.Marker({
            map: than.map,
            zIndex: 100, // 层级
            extData: {
              // 携带数据
              id: i,
              lnglat: arr[i].lnglat,
              lnglatRange: arr[i].lnglatRange,
              name: arr[i].name
            },
            position: arr[i].lnglat,
            // label: {
            //   content: `<div style="background-color: white;">${arr[i].name}</div>`,
            //   offset: new AMap.Pixel(5, 0),
            //   direction: "right"
            // },
            content: `<div style="position: relative;width: 200px;padding:3px 10px;border-radius: 10px;background-color: rgba(0, 167, 91, 0.9);color: white;text-align: center;">${arr[i].name}<div style="width:0px;height:0px;border-width:8px 8px 0px 8px;border-style:solid;border-color:rgba(0, 167, 91, 0.9) transparent transparent transparent;position:absolute;buttom:0;left:50%;"></div></div>`,
            offset: new AMap.Pixel(-13, -30)
          });
          than.markerList.push(than.marker);
        }
      } else if (val === "板块" || val === "商圈") {
        for (let i = 0; i < arr.length; i++) {
          than.marker = new AMap.Marker({
            map: than.map,
            zIndex: 100, // 层级
            // anchor: 'top-center',
            extData: {
              // 携带数据
              id: i,
              lnglatRange: arr[i].lnglatRange,
              name: arr[i].name
            },
            position: arr[i].lnglat,
            content: `<div style="position: relative;width: 200px;padding:3px 10px;border-radius: 10px;background-color: rgba(0, 167, 91, 0.9);color: white;text-align: center;">${arr[i].name}</div>`,
            offset: new AMap.Pixel(-13, -30)
          });
          than.markerList.push(than.marker);
        }
      } else if (val === "小区板块"){
        for (let i = 0; i < arr.length; i++) {
          than.count = than.countFn(than.buttonState, i, arr[i].name); // 统计板块下面的小区数
          than.marker = new AMap.Marker({
            map: than.map,
            zIndex: 100, // 层级
            extData: {
              // 携带数据
              id: i,
              lnglatRange: arr[i].lnglatRange,
              name: arr[i].name,
              count: than.count
            },
            position: arr[i].lnglat,
            content: `<div style="position: relative;width: 200px;padding:3px 10px;border-radius: 10px;background-color: rgba(0, 167, 91, 0.9);color: white;text-align: center;"><div>${arr[i].name}</div><div>${than.buttonStateString}：${than.count}个</div></div>`,
            offset: new AMap.Pixel(-13, -30)
          });
          than.markerList.push(than.marker);
        }
      }else {
        for (let i = 0; i < arr.length; i++) {
          than.count = than.countFn(than.buttonState, i); // 统计行政区下面的板块或商圈或小区数
          than.marker = new AMap.Marker({
            map: than.map,
            zIndex: 100, // 层级
            extData: {
              // 携带数据
              id: i,
              lnglatRange: arr[i].lnglatRange,
              name: arr[i].name,
              count: than.count
            },
            position: arr[i].lnglat,
            content: `<div style="position: relative;width: 90px;height: 90px;border-radius: 50%;background-color: rgba(0, 167, 91, 0.9);color: white;text-align: center;padding-top:20px;overflow: hidden;"><div style="margin-bottom:5px;">${arr[i].name}</div><div>${than.buttonStateString}：${than.count}个</div></div>`,
            offset: new AMap.Pixel(-13, -30)
          });
          than.markerList.push(than.marker);
        }
      }
    },
    // 鼠标点击覆盖物
    showInfoC: function(e) {
      console.log(e);
      if (
        than.nowMapState === 0 ||
        (than.nowMapState === 1 && than.buttonState === 0)
      ) {
        var id = e.target.B.extData.id;
        var name = e.target.B.extData.name;
        if (than.nowMapState === 0) {
          than.map.setFitView(than.markerList[id], true, [0, 0, 0, 0], 13);
        } else if (than.nowMapState === 1 && than.buttonState === 0) {
          than.map.setFitView(than.markerList[id], true, [0, 0, 0, 0], 15);
        }
        than.updateMarker(id, name, than.nowMapState, than.buttonState);
        than.nowMapState++;
      } else {
        // ...跳转
        if(than.buttonState === 0){
          localStorage.setItem('id0', than.id0); // 保存所选行政区id
          localStorage.setItem('id1', than.id1); // 保存所选小区板块id
          localStorage.setItem('buttonState', '0');
          this.$router.push({path: '/non_visitor_new_house'})
        }else if(than.buttonState === 1){
          localStorage.setItem('id0', than.id0); // 保存所选行政区id
          localStorage.setItem('buttonState', '1');
          this.$router.push({path: '/non_visitor_new_commercialOperation'})
        }else if(than.buttonState === 2){
          localStorage.setItem('id0', than.id0); // 保存所选行政区id
          localStorage.setItem('buttonState', '2');
          this.$router.push({path: '/non_visitor_new_estate'})
        }
      }
    },
    // 鼠标进入覆盖物
    showInfoOver: function(e) {
      than.changeMarkerState(e);
      // 地图处于“行政区”层
      if (than.nowMapState === 0) {
        switch (e.target.B.extData.id) {
          case 0:
            than.drawBounds("500103");
            break;
          case 1:
            than.drawBounds("500106");
            break;
          case 2:
            than.drawBounds("500108");
            break;
          case 3:
            than.drawBounds("500107");
            break;
          case 4:
            than.drawBounds("500105");
            break;
          case 5:
            than.drawBounds("500109");
            break;
          case 6:
            than.drawBounds("500113");
            break;
          case 7:
            than.drawBounds("500104");
            break;
          case 8:
            than.drawBounds("500112");
            break;
          default:
            break;
        }
      }
      // 地图进入“板块或商圈”、且按钮状态为“居民”
      else if (than.nowMapState === 1 && than.buttonState === 0) {
        var polygon = new AMap.Polygon({
          map: than.map,
          path: e.target.B.extData.lnglatRange, //设置多边形边界路径
          strokeWeight: 1,
          strokeColor: "#0091ea",
          fillColor: "#80d8ff",
          fillOpacity: 0.4 //填充透明度
        });
        than.polygons.push(polygon);
      }
      // 地图进入“板块或商圈”、且按钮状态为“板块”
      else if (than.nowMapState === 1 && than.buttonState === 2) {
        var polygon = new AMap.Polygon({
          map: than.map,
          path: e.target.B.extData.lnglatRange, //设置多边形边界路径
          strokeWeight: 1,
          strokeColor: "#0091ea",
          fillColor: "#80d8ff",
          fillOpacity: 0.4 //填充透明度
        });
        than.polygons.push(polygon);
      }
      // 地图进入“板块或商圈”、且按钮状态为“商圈”
      else if (than.nowMapState === 1 && than.buttonState === 1) {
        var polygon = new AMap.Polygon({
          map: than.map,
          path: e.target.B.extData.lnglatRange, //设置多边形边界路径
          strokeWeight: 1,
          strokeColor: "#0091ea",
          fillColor: "#80d8ff",
          fillOpacity: 0.4 //填充透明度
        });
        than.polygons.push(polygon);
      } else if (than.nowMapState === 2) {
        // 地图进入“小区”
      }
    },
    // 鼠标移出覆盖物
    showInfoOut: function(e) {
      than.regainMarkerState(e);
      than.map.remove(than.polygons);
    },
    // 更换覆盖物
    updateMarker: function(id, name, nowMapState, buttonState) {
      // 删除原有覆盖物
      than.map.remove(than.markerList);
      than.map.remove(than.polygons);
      // 添加新的覆盖物
      if (nowMapState === 0 && buttonState === 0) {
        than.id0 = id;
        than.addMarker(than.estateArr[id].estateArr, "小区板块");
      } else if (nowMapState === 0 && buttonState === 1) {
        than.id0 = id;
        than.addMarker(than.commercialArr[id].commercialArr, "商圈");
      } else if (nowMapState === 0 && buttonState === 2) {
        than.id0 = id;
        than.addMarker(than.estateArr[id].estateArr, "板块");
      } else if (nowMapState === 1 && buttonState === 0) {
        // ...添加小区覆盖物
        than.id1 = id;
        for (let i = 0; i < than.houseArr.length; i++) {
          if (than.houseArr[i].name === name) {
            than.addMarker(than.houseArr[i].houseArr, "小区");
            than.restaurants = than.houseArr[i].houseArr;
            break;
          }
        }
      }
      than.Event();
    },
    // 地图缩放
    mapScaling: function() {
      // 禁止页面过度缩小
      if (than.map.getZoom() < 11) {
        than.map.setZoom(11);
        than.map.setCenter(than.center);
      }
      // 禁止页面过度放大
      if (than.map.getZoom() > 15) {
        // than.map.setZoom(15);
      }
      // 通过页面缩小、控制不同覆盖物显示
      if(than.nowMapState===2 && than.buttonState===0){
        // 小区
        if(than.map.getZoom() <= 13){
          // console.log('小区13')
          than.nowMapState--;
          than.updateMarker(than.id0, '' ,than.nowMapState - 1 ,than.buttonState);
        }
      }else if(than.nowMapState===1 && than.buttonState===0){
        // 小区板块
        if(than.map.getZoom()===11){
          // console.log('小区板块11')
          than.nowMapState--;
          than.map.remove(than.markerList);
          than.map.remove(than.polygons);
          than.addMarker(than.cantonArr);
          than.Event();
        }
      }else if(than.nowMapState===1 && than.buttonState===1){
        // 商圈
        if(than.map.getZoom()===11){
          // console.log('商圈11')
          than.nowMapState--;
          than.map.remove(than.markerList);
          than.map.remove(than.polygons);
          than.addMarker(than.cantonArr);
          than.Event();
        }
      }else if(than.nowMapState===1 && than.buttonState===2){
        // 板块
        if(than.map.getZoom()===11){
          // console.log('板块11')
          than.nowMapState--;
          than.map.remove(than.markerList);
          than.map.remove(than.polygons);
          than.addMarker(than.cantonArr);
          than.Event();
        }
      }
    },
    // 绑定事件
    Event: function() {
      // 给覆盖物绑定
      for (let i = 0; i < than.markerList.length; i++) {
        than.markerList[i].on("click", than.showInfoC);
        than.markerList[i].on("mouseover", than.showInfoOver);
        than.markerList[i].on("mouseout", than.showInfoOut);
      }
      // 给地图绑定缩放事件
      than.map.on("zoomend", than.mapScaling);
    },
    // 改变覆盖物层级和颜色
    changeMarkerState: function(e) {
      if (than.nowMapState === 2) {
        than.markerList[e.target.B.extData.id].setzIndex(200); // 显示在最上层
        // than.markerList[e.target.B.extData.id].setLabel({
        //   content: `<div style="background-color: white;color: blue;">${e.target.B.extData.name}</div>`,
        //   offset: new AMap.Pixel(5, 0),
        //   direction: "right"
        // });
        than.markerList[e.target.B.extData.id].setContent(
          `<div style="position: relative;width: 200px;padding:3px 10px;border-radius: 10px;background-color: rgba(243, 49, 49, 0.9);color: white;text-align: center;">${e.target.B.extData.name}<div style="width:0px;height:0px;border-width:8px 8px 0px 8px;border-style:solid;border-color:rgba(243, 49, 49, 0.9) transparent transparent transparent;position:absolute;buttom:0;left:50%;"></div></div>`
        );
      } else if (than.nowMapState===1 && than.buttonState===0) {
        than.markerList[e.target.B.extData.id].setzIndex(200); // 显示在最上层
        than.markerList[e.target.B.extData.id].setContent(
          `<div style="position: relative;width: 200px;padding:3px 10px;border-radius: 10px;background-color: rgba(243, 49, 49, 0.9);color: white;text-align: center;"><div>${e.target.B.extData.name}</div><div>${than.buttonStateString}：${e.target.B.extData.count}个</div></div>`
        );
      }else if (than.nowMapState === 1) {
        than.markerList[e.target.B.extData.id].setzIndex(200); // 显示在最上层
        than.markerList[e.target.B.extData.id].setContent(
          `<div style="position: relative;width: 200px;padding:3px 10px;border-radius: 10px;background-color: rgba(243, 49, 49, 0.9);color: white;text-align: center;">${e.target.B.extData.name}</div>`
        );
      } else {
        than.markerList[e.target.B.extData.id].setzIndex(200); // 显示在最上层
        than.markerList[e.target.B.extData.id].setContent(
          `<div style="position: relative;width: 90px;height: 90px;border-radius: 50%;background-color: rgba(243, 49, 49, 0.9);color: white;text-align: center;padding-top:20px;overflow: hidden;"><div style="margin-bottom:5px;">${e.target.B.extData.name}</div><div>${than.buttonStateString}：${e.target.B.extData.count}个</div></div>`
        );
      }
    },
    // 恢复覆盖物层级和颜色
    regainMarkerState: function(e) {
      if (than.nowMapState === 2) {
        than.markerList[e.target.B.extData.id].setzIndex(100);
        // than.markerList[e.target.B.extData.id].setLabel({
        //   content: `<div style="background-color: white;">${e.target.B.extData.name}</div>`,
        //   offset: new AMap.Pixel(5, 0),
        //   direction: "right"
        // });
        than.markerList[e.target.B.extData.id].setContent(
          `<div style="position: relative;width: 200px;padding:3px 10px;border-radius: 10px;background-color: rgba(0, 167, 91, 0.9);color: white;text-align: center;">${e.target.B.extData.name}<div style="width:0px;height:0px;border-width:8px 8px 0px 8px;border-style:solid;border-color:rgba(0, 167, 91, 0.9) transparent transparent transparent;position:absolute;buttom:0;left:50%;"></div></div>`
        );
      } else if (than.nowMapState===1 && than.buttonState===0) {
        than.markerList[e.target.B.extData.id].setzIndex(100);
        than.markerList[e.target.B.extData.id].setContent(
          `<div style="position: relative;width: 200px;padding:3px 10px;border-radius: 10px;background-color: rgba(0, 167, 91, 0.9);color: white;text-align: center;"><div>${e.target.B.extData.name}</div><div>${than.buttonStateString}：${e.target.B.extData.count}个</div></div>`
        );
      }else if (than.nowMapState === 1) {
        than.markerList[e.target.B.extData.id].setzIndex(100);
        than.markerList[e.target.B.extData.id].setContent(
          `<div style="position: relative;width: 200px;padding:3px 10px;border-radius: 10px;background-color: rgba(0, 167, 91, 0.9);color: white;text-align: center;">${e.target.B.extData.name}</div>`
        );
      } else {
        than.markerList[e.target.B.extData.id].setzIndex(100);
        than.markerList[e.target.B.extData.id].setContent(
          `<div style="position: relative;width: 90px;height: 90px;border-radius: 50%;background-color: rgba(0, 167, 91, 0.9);color: white;text-align: center;padding-top:20px;overflow: hidden;"><div style="margin-bottom:5px;">${e.target.B.extData.name}</div><div>${than.buttonStateString}：${e.target.B.extData.count}个</div></div>`
        );
      }
    },
    // 统计板块/商圈/小区的个数
    countFn: function(buttonState, id, name){
      let count = 0;
      if(buttonState===2){
        // 行政区板块数
        count = than.estateArr[id].estateArr.length;
      }else if(buttonState===1){
        // 行政区商圈数
        count = than.commercialArr[id].commercialArr.length;
      }else if(buttonState===0 && name){
        // 板块小区数
        for(let j=0; j<than.houseArr.length; j++){
          if(name === than.houseArr[j].name){
              count = than.houseArr[j].houseArr.length;
          }
        }
      }else{
        // 行政区小区数
        for(let i=0; i<than.estateArr[id].estateArr.length; i++){
          for(let j=0; j<than.houseArr.length; j++){
            if(than.estateArr[id].estateArr[i].name === than.houseArr[j].name){
                count = than.houseArr[j].houseArr.length;
            }
          }
        }
      }
      return count;
    }
  },
  created() {
    than = this;
    let buttonState =  localStorage.getItem('buttonState') || 2;
    than.buttonState =  parseInt(buttonState);
  },
  mounted() {
    than.map = new AMap.Map("container", {
      resizeEnable: true,
      features: ["bg", "building", "road"], // "point"
      center: than.center,
      zoom: 11, //地图显示的缩放级别
      isHotspot: false
    });
    than.addMarker(than.cantonArr);
    than.Event();

    // 从图表页面点击返回按钮返回
    if(true){
      if(localStorage.getItem('id0') && localStorage.getItem('id1')){
        // 渲染小区
        than.id0 =  localStorage.getItem('id0');
        than.id1 =  localStorage.getItem('id1');
        than.nowMapState = 1;
        than.map.setFitView(than.markerList[than.id0], true, [0, 0, 0, 0], 13);
        than.updateMarker(than.id0, '' , than.nowMapState - 1 ,than.buttonState);
        than.nowMapState = 2;
        than.map.setFitView(than.markerList[than.id1], true, [0, 0, 0, 0], 15);
        than.updateMarker(than.id1, than.markerList[than.id1].B.extData.name , than.nowMapState - 1 ,than.buttonState);
      }else if(localStorage.getItem('id0')){
        // 渲染板块或商圈
        than.id0 =  localStorage.getItem('id0');
        than.nowMapState = 1;
        than.map.setFitView(than.markerList[than.id0], true, [0, 0, 0, 0], 13);
        than.updateMarker(than.id0, '' , than.nowMapState - 1 ,than.buttonState);
      }
      localStorage.removeItem('id0');
      localStorage.removeItem('id1');
    }
  }
};
</script>

<style lang="scss" scoped>
#container {
  margin: 0;
  min-height: calc(100vh - 50px);
}
.search {
  width: 200px;
  position: absolute;
  right: 15px;
  top: 15px;
  z-index: 2;
}
</style>