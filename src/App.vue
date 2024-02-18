<script setup>
import { onMounted, onUnmounted, ref } from 'vue'

//引入SuperAPI
import cloudRenderer from '51superapi'
const app = new cloudRenderer('player')

const showLoading = ref(true)

const myStartRender = async (width, height) => {
  //配置参数
  const startRenderConfig = {
    url: 'https://vizservice-paas.51aes.com', //[必须] 云渲染服务地址; 8889:固定端口
    order: '65c0adb60719f9defeb925f4', //[必须] 渲染口令; 在云渲染客户端上获得
    resolution: [width, height], //[可选] 设置渲染场景像素分辨率
    nodestyle: 'position:absolute;border:1px solid #09f;', //[可选] 设置渲染场景容器DOM节点样式, 与设置渲染场景像素分辨率配对使用
    keyboard: 'keyboardnofn', //[可选] 初始建盘事件, 开启wasd方向键 [选项: keyboard/keyboardnofn; 详见注册键盘事件]
    setlogmode: true //[可选] 开启/关闭SuperAPI调用日志, 默认false
  }
  try {
    await app.startRender(startRenderConfig).then((el) => {
      console.log('打印信息', el)

      // 事件注册；事件监听处理器函数, 接收所有从云渲染返回的事件, 数据等信息
      app.RegisterCloudResponse(myHandleResponseFunction)
    })
  } catch (error) {
    console.error('error:', error)
  }
}

// 事件注册函数
const myHandleResponseFunction = (data) => {
  const jsonObject = typeof data === 'object' ? JSON.parse(JSON.stringify(data)) : JSON.parse(data)
  switch (jsonObject.func_name) {
    case 'APIAlready':
      showLoading.value = false
      console.log('APIAlready: 3D世界加载完成')
      break
  }
  return data
}
//添加poi
const addPOI = async () => {
  const poiJson = {
    id: 'poi_id',
    label: '这是市中心人民广场！',
    coord_type: 0,
    cad_mapkey: '',
    coord: '113.335716,23.129055',
    coord_z: 0,
    coord_z_type: 0,
    always_show_label: true,
    show_label_range: '0,2000',
    umg_type: 'default',
    sort_order: true
  }
  const addPoiRes = await app.SuperAPI('AddPOI', poiJson)
  console.log('添加poi', addPoiRes) //成功、失败回调

  const cameraJson = {
    camera_id: 'camera1',
    coord_type: '0', //位置的坐标类型(0:经纬度坐标, 1:cad坐标)
    cad_mapkey: '', //CAD基准点Key值, 项目中约定
    coord: '113.335716,23.129055', //相机位置坐标 lng, lat
    coord_z: 35, //高度(单位:米)
    coord_z_type: 0, //坐标高度类型(0:相对3D世界表面；1:相对3D世界地面；2:相对3D世界海拔; 注:cad坐标无效)
    //若需要根据“获取镜头信息”API回调信息中的高度参数设置相同位姿的镜头，此处需要选择“2”
    pitch: 30, //镜头俯仰角(-90~90)
    pitch_limit: '0,90', //镜头俯仰角范围(-90~90)
    yaw: 0, //镜头偏航角(0正北, 0~359)
    yaw_limit: '0,359', //镜头偏航角范围(0正北, 0~359)
    arm_distance: 300, //镜头距坐标点距离(单位:米)
    arm_distance_limit: '50,10000' //镜头距坐标点距离范围(单位:米)
  }
  const addCameraRes = await app.SuperAPI('AddCamera', cameraJson)
  console.log('添加镜头', addCameraRes)

  const setCameraJson = {
    camera_id: 'camera1',
    fly: true //true, false
  }
  app.SuperAPI('SetCameraActive', setCameraJson)
}

//添加特效
const addEffect = async () => {
  const jsondata = {
    id: 'city_fire',
    coord_type: 0, //坐标类型(0:经纬度坐标, 1:cad坐标)
    cad_mapkey: '', //CAD基准点Key值, 项目中约定
    type: 'flame', //样式类型(见下表)
    scale: 200, //半径(单位:米; "adaptive":true 时含义为倍率)
    adaptive: false, //true:自适应大小;false:默认
    coord: '113.361038,23.155542', //坐标点 lng,lat
    coord_z: 3, //高度(单位:米)
    coord_z_type: 0, //坐标高度类型(0:相对3D世界表面；1:相对3D世界地面；2:相对3D世界海拔; 注:cad坐标无效)
    pitch: 0, //俯仰角, 参考(-90~90)
    roll: 0, //翻滚角, 参考(0~360)
    yaw: 0, //偏航角, 参考(0正北, 0~360)
    title_text: '', //文本内容, 富文本内容
    title_offset: '10,10', //文本偏移(单位:米; 东西向为x轴进行偏移)
    title_face_to_camera: false, //顶部文字是否跟踪朝向摄像机(注: true优先, "pitch", "roll", "yaw" 无效)
    title_text_portrait: false //顶部文字排列方向(true: 纵向, false: 横向)
  }
  const addEffectRes = await app.SuperAPI('AddEffect', jsondata)
  console.log(addEffectRes)

  const cameraJson = {
    camera_id: 'camera2',
    coord_type: '0', //位置的坐标类型(0:经纬度坐标, 1:cad坐标)
    cad_mapkey: '', //CAD基准点Key值, 项目中约定
    coord: '113.361038,23.155542', //相机位置坐标 lng, lat
    coord_z: 13.84, //高度(单位:米)
    coord_z_type: 0, //坐标高度类型(0:相对3D世界表面；1:相对3D世界地面；2:相对3D世界海拔; 注:cad坐标无效)
    //若需要根据“获取镜头信息”API回调信息中的高度参数设置相同位姿的镜头，此处需要选择“2”
    pitch: 30, //镜头俯仰角(-90~90)
    pitch_limit: '0,90', //镜头俯仰角范围(-90~90)
    yaw: 0, //镜头偏航角(0正北, 0~359)
    yaw_limit: '0,359', //镜头偏航角范围(0正北, 0~359)
    arm_distance: 300, //镜头距坐标点距离(单位:米)
    arm_distance_limit: '50,10000' //镜头距坐标点距离范围(单位:米)
  }
  const addCameraRes = await app.SuperAPI('AddCamera', cameraJson)
  console.log('添加镜头', addCameraRes)

  const setCameraJson = {
    camera_id: 'camera2',
    fly: true //true, false
  }
  app.SuperAPI('SetCameraActive', setCameraJson)
}

const addPath = () => {
  const jsondata = {
    id: 'Path_id', //路径id
    advancedSetting: {
      smoothnessOfCorners: 'extremelyHigh' //设置路径边角的平滑度(extremelyHigh:极高; high:高; middle:中; low:低;)
    },
    coord_type: 0, //坐标类型(0:经纬度坐标, 1:cad坐标)
    cad_mapkey: 'default', //CAD基准点Key值, 项目中约定
    coord_z_type: 0, //坐标高度类型(0:相对3D世界表面; 1:相对3D世界地面; 2:相对3D世界海拔; 注:cad坐标无效)
    type: 'fit_solid', //路径样式类型: fit_solid(贴合地面), adaptive_solid(等宽路径),none, solid, arrow, arrow_dot, dashed_dot, arrow_dashed, flash, scan_line, brimless_arrow, railway, round_pipe, square_pipe, dashed_line
    color: 'ff00ff', //路径颜色(HEX颜色值, 空值即透明; railway类型无效, 默认黑白色)
    pass_color: 'ff0000', //覆盖物移动经过路径颜色(HEX颜色值)
    width: 35, //宽度(单位:米, 圆柱直径或方柱边长; 当类型为"adaptive_solid", 含义为倍率)
    speedRate: 5, //用于水流倍率(取值范围[0,10])
    points: [
      //路径点数据(必须有顺序)
      {
        coord: '113.34372452,23.15007070', //路径点的坐标
        coord_z: 20 //路径点高度(单位:米)
      },
      {
        coord: '113.33174508,23.14740030',
        coord_z: 20
      },
      {
        coord: '113.33644726,23.15753133',
        coord_z: 20
      },
      {
        coord: '113.32825861,23.12480244',
        coord_z: 20
      },
      {
        coord: '113.34894067,23.13632341',
        coord_z: 20
      }
    ]
  }

  app.SuperAPI('RemoveAllCovering', { covering_type: 'path' })

  app.SuperAPI('AddPath', jsondata, (_back) => {
    console.log(_back)
    let data = JSON.parse(_back)
    if (data.success) {
      setTimeout(() => app.SuperAPI('FocusAllCovering', { covering_type: 'path' }), 5e2)
    }
  })
}

const addHeatMap = () => {
  const jsondata = {
    id: 'heatmap_id', //区域热力图id
    coord_type: 0, //区域热力图中坐标类型(0:经纬度坐标, 1:cad  坐标)
    cad_mapkey: 'default', //CAD基准点Key值, 项目中约定
    coord_z: 35, //高度(单位:米, cad坐标无效)
    coord_z_type: 0, //坐标高度类型(0:相对3D世界表面; 1:相对3D世界地面; 2:相对3D世界海拔; 注:cad坐标无效)
    heatmap_type: 1, //区域热力图样式类型(1: 投影型 2:平面型)
    leftupper_coord: '113.33337166,23.11540602', //热力区域坐标(左上坐标 lng,lat)
    leftlower_coord: '113.32565440,23.11516571', //热力区域坐标(左下坐标 lng,lat)
    rightlower_coord: '113.34372452,23.15007070', //热力区域坐标(右下坐标 lng,lat)
    rightupper_coord: '113.33174508,23.14740030', //热力区域坐标(右上坐标 lng,lat)
    brush_diameter: 1000, //热力点笔刷直径(单位:米, 单个热力点覆盖直径)
    heatpoint_minvalue: 1, //热力点热力值范围最小值(1:绿色接近透明, 100:最红, 线性计算)
    heatpoint_maxvalue: 100, //热力点热力值范围最大值
    gradient_setting: {
      //自定义热力点渐变颜色(HEX颜色值,共5个色值)
      color1: '00ffff',
      color2: '00ff00',
      color3: 'ffff00',
      color4: 'ff00ff',
      color5: 'ff0000'
    },
    data: [
      {
        coord: '113.31982596,23.13509584',
        value: 38
      },
      {
        coord: '113.32967503,23.15792498',
        value: 55
      },
      {
        coord: '113.33337166,23.11540602',
        value: 73
      },
      {
        coord: '113.32565440,23.11516571',
        value: 64
      },
      {
        coord: '113.34372452,23.15007070',
        value: 39
      },
      {
        coord: '113.33174508,23.14740030',
        value: 65
      },
      {
        coord: '113.33644726,23.15753133',
        value: 67
      },
      {
        coord: '113.32825861,23.12480244',
        value: 43
      },
      {
        coord: '113.34894067,23.13632341',
        value: 57
      },
      {
        coord: '113.33261630,23.12858740',
        value: 71
      },
      {
        coord: '113.35594288,23.14516342',
        value: 49
      },
      {
        coord: '113.32510181,23.14546100',
        value: 49
      },
      {
        coord: '113.32943038,23.12624447',
        value: 71
      },
      {
        coord: '113.32234523,23.13667736',
        value: 25
      },
      {
        coord: '113.33689710,23.13953716',
        value: 62
      },
      {
        coord: '113.35714095,23.13006622',
        value: 14
      },
      {
        coord: '113.32014876,23.11796036',
        value: 45
      },
      {
        coord: '113.34079433,23.14970048',
        value: 16
      },
      {
        coord: '113.32669627,23.14552538',
        value: 12
      },
      {
        coord: '113.33805626,23.15422779',
        value: 28
      },
      {
        coord: '113.32635794,23.13200846',
        value: 22
      },
      {
        coord: '113.32437462,23.14819967',
        value: 57
      },
      {
        coord: '113.31648698,23.12837277',
        value: 23
      },
      {
        coord: '113.33604088,23.11940896',
        value: 24
      },
      {
        coord: '113.34498006,23.15620064',
        value: 69
      }
    ]
  }

  app.SuperAPI('RemoveAllCovering', { covering_type: 'heatmap' })

  app.SuperAPI('AddHeatMap', jsondata)

  setTimeout(() => app.SuperAPI('FocusAllCovering', { covering_type: 'heatmap' }), 5e2)
}

const addRange = () => {
  const jsondata = {
    id: 'range_id', //区域id
    coord_type: 0, //坐标类型(0:经纬度坐标, 1:cad坐标)
    cad_mapkey: 'default', //CAD基准点Key值, 项目中约定
    coord_z: 0, //高度(单位:米, cad坐标无效)
    coord_z_type: 0, //坐标高度类型(0:相对3D世界表面; 1:相对3D世界地面; 2:相对3D世界海拔; 注:cad坐标无效)
    type: 'grid', //样式类型(none, wave, grid, loop_line, stripe, bias, [ box_solid, box_solid_line, box_wave, box_wave_line 选此类型时, 底部区域填充fill_area无效 ])
    color: '0000ff4c', //颜色(HEXA颜色值)
    range_height: 150, //围栏高度(单位:米)
    fill_area: 'dot', //底部填充类型(none, solid, block, block2, dot, dot2, dot3, dash_line, radar)
    stroke_weight: 10, //底部轮廓线宽度(单位:米; 注:区域中含有内环"inner_points"时无效)
    points: [
      {
        coord: '113.31577481,23.14558678'
      },
      {
        coord: '113.35710315,23.13431391'
      },
      {
        coord: '113.34711188,23.12025466'
      }
    ]
  }

  app.SuperAPI('RemoveAllCovering', { covering_type: 'range' })

  app.SuperAPI('AddRange', jsondata, (_back) => {
    console.log(_back)
  })

  setTimeout(() => app.SuperAPI('FocusAllCovering', { covering_type: 'range' }), 5e2)
}

onMounted(() => {
  myStartRender(window.innerWidth, window.innerHeight) //启动云渲染
})

onUnmounted(() => {
  app.StopRenderCloud() //关闭云渲染, 释放资源
})
</script>

<template>
  <div id="player" style="display: none">
    <div class="btn-wrap" v-if="!showLoading">
      <button class="btn" @click="addPOI">定位到市中心</button>
      <button class="btn" @click="addEffect">模拟火灾特效</button>
      <button class="btn" @click="addPath">添加路径</button>
      <button class="btn" @click="addHeatMap">添加热力图</button>
      <button class="btn" @click="addRange">添加区域轮廓</button>
    </div>
  </div>
  <div v-if="showLoading" class="load">loading...</div>
</template>

<style scoped>
.load {
  font-size: 30px;
  color: blue;
  /* margin: 0 auto; */
  text-align: center;
}
.btn-wrap {
  position: absolute;
  top: 50px;
  left: 20px;
  z-index: 10;
  display: flex;
  flex-direction: column;
  .btn {
    margin-bottom: 10px;
  }
}
</style>
