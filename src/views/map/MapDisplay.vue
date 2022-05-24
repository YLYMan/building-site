<template>
	<!-- <div id="mapDiv" style="position:absolute;width:100%; height:100%"></div> -->
	<div id="mapDiv">
		<button
			style="position: absolute;
			z-index: 500;
			border: 1px solid;
			right: 7%;
			bottom: 10%;
			color: red;
			background: green;
			padding: 10px;"
      @click="positionBtn"
    >定位坐标</button>
		<button
			style="position: absolute;
			z-index: 500;
			border: 1px solid;
			right: 7%;
			bottom: 5%;
			color: red;
			background: green;
			padding: 10px;"
			@click="pointsBtn"
		>添加多个点</button>
	</div>
</template>

<script>
// import { map } from 'store/storages/all'
const TMAP_NORMAL_MAP = 'TMAP_NORMAL_MAP' // 普通街道视图
// const TMAP_SATELLITE_MAP = 'TMAP_SATELLITE_MAP' // 卫星视图
const TMAP_HYBRID_MAP = 'TMAP_HYBRID_MAP' // 卫星和路网的混合视图
// const TMAP_TERRAIN_MAP = 'TMAP_TERRAIN_MAP' // 地形视图
// const TMAP_TERRAIN_HYBRID_MAP = 'TMAP_TERRAIN_HYBRID_MAP' // 地形和路网的混合视图

export default {
	data () {
		return {
			cityName: '' // 暂存城市名称
		}
	},
	methods: {
		onLoad () {
			const T = window.T // 防止报错
			const that = this
			const zoom = 12
			that.map = new T.Map('mapDiv')
			that.map.centerAndZoom(new T.LngLat(116.40769, 39.89945), zoom)
			const ctrl = new T.Control.MapType([
				{
					title: '地图', // 地图控件上所要显示的图层名称
					icon: 'http://api.tianditu.gov.cn/v4.0/image/map/maptype/vector.png', // 地图控件上所要显示的图层图标（默认图标大小80x80）
					layer: TMAP_NORMAL_MAP // 地图类型对象，即MapType。
				},
				{
					title: '卫星混合',
					icon:
						'http://api.tianditu.gov.cn/v4.0/image/map/maptype/satellitepoi.png',
					layer: TMAP_HYBRID_MAP
				}
			])
      that.map.addControl(ctrl)
		},
		GetMaps () {
			const T = window.T

			// 设置显示地图的中心点和级别
			this.map.clearOverLays()
			this.map.centerAndZoom(new T.LngLat(116.40769, 39.89945), 12)

			const icon = new T.Icon({
				iconUrl: 'http://api.tianditu.gov.cn/img/map/markerA.png',
				iconSize: new T.Point(33, 33),
				iconAnchor: new T.Point(10, 25)
			})

			const latlng = new T.LngLat(116.411794, 39.9068) // 设置坐标点传入经度纬度
			const marker = new T.Marker(latlng, { icon: icon }) // 创建标注
			this.map.addOverLay(marker) // 向地图上添加标注

			const that = this
			marker.addEventListener('click', function (e) { // 监听点击事件
				console.log(e)
				const clientx = e.lnglat.lat // 获取marker当前经纬度
				const clientY = e.lnglat.lng
				that.map.centerAndZoom(new T.LngLat(clientY, clientx), 12) // 重新创建地图对象
				// 这里获取的经度纬度是实际经纬度四舍五入后的获取的
			})
		},

		// 定位用户坐标，绘制城市边界，并通过移动标点重新定位经纬度
		positionBtn () {
			const T = window.T
      const that = this

      // this.map.clearOverLays() // 清理地图上的覆盖物

			// IP城市定位
      const lc = new T.LocalCity() // 创建一个获取本地城市位置的实例
      lc.location(function (e) { // 利用ip进行定位
        alert(e.cityName)
        const marker = new T.Marker(e.lnglat) // e.lnglat，标注点的地理坐标
        that.map.addOverLay(marker) // addOverLay方法，将覆盖物添加到地图中，一个覆盖物实例只能向地图中添加一次。
        that.getMap() // 创建地图对象

        const overlayStyle = e => {
          const p = e.target
          alert(
            '该地区经纬度是：' + p.getLngLat().lng + ',' + p.getLngLat().lat
          )
        }

        marker.addEventListener('dragend', overlayStyle) // 添加事件监听函数。
        marker.enableDragging() // 开启标注拖拽功能

			// H5定位
			// const lo = new T.Geolocation()
			// const fn = e => {
			// 		if (this.getStatus() === 0) {
			// 				that.map.centerAndZoom(e.lnglat, 15)
			// 				alert('获取定位坐标：' + e.lnglat.lat + ',' + e.lnglat.lng)
			// 				const marker = new T.Marker(e.lnglat)
			// 				that.map.addOverLay(marker)
			// 		}
			// 		if (this.getStatus() === 1) {
			// 				that.map.centerAndZoom(e.lnglat, e.level)
			// 				alert('获取定位坐标：' + e.lnglat.lat + ',' + e.lnglat.lng)
			// 				const marker = new T.Marker(e.lnglat)
			// 				that.map.addOverLay(marker)
			// 		}
			// }
			// lo.getCurrentPosition(fn)
		},
    getMap () {
			const T = window.T
      const that = this
      // 创建对象
      const administrative = new T.AdministrativeDivision() // 创建一个获取行政区划的实例
      const config = {
        needSubInfo: false, // 是否需要下一级信息
        needAll: false, // 是否需要所有子节点。
        needPolygon: true, // 是否需要行政区划范围。
        needPre: true, // 是否需要上一级所有信息。
        searchType: 1, // 查询类型。0表示根据code查询，1表示根据名称查询。
        searchWord: this.cityName // 查询行政区划的名称。
      }
      administrative.search(config, searchResult) // 方法：根据检索词发起检索，searchResult：回调参数
      function searchResult (result) {
        if (result.getStatus() === 100) {
          const data = result.getData()
          that.showMsg(data)
        } else {
          result.getMsg()
        }
      }
      // 具体内容详见AdministrativeDivisionResult类。
    },
    showMsg (data) {
      for (let i = 0; i < data.length; i++) {
        // 解释上级行政区划
        if (data[i].parents) {
          let upLevel = ''
          if (data[i].parents.country) {
            upLevel += data[i].parents.country.name
          }
          if (data[i].parents.province) {
            upLevel += data[i].parents.province.name
          }
					console.log(upLevel)
        }

        if (data[i].points) {
          // 绘制行政区划
          this.polygon(data[i].points)
        }

        // 解释下级行政区划
        if (data[i].child) {
          this.showMsg(data[i].child)
          console.log(data[i].child.points)
          if (data[i].child.points) {
            // 绘制行政区划
            this.polygon(data[i].child.points)
          }
        }
      }
    },
    polygon (points) {
			const T = window.T
      const that = this
      const pointsArr = []
      for (let i = 0; i < points.length; i++) {
        const regionLngLats = []
        const regionArr = points[i].region.split(',')
        for (let m = 0; m < regionArr.length; m++) {
          const lnglatArr = regionArr[m].split(' ')
          const lnglat = new T.LngLat(lnglatArr[0], lnglatArr[1])
          regionLngLats.push(lnglat)
          pointsArr.push(lnglat)
        }
        // 创建面对象,样式
        const polygon = new T.Polygon(regionLngLats, {
          color: '#fd737a',
          weight: 3,
          opacity: 1,
          fillColor: '#FFFFFF', // 填充颜色。
          fillOpacity: 0.3 // 填充透明度
        })
        // 向地图上添加行政区划面
        that.map.addOverLay(polygon)
      }
      // 显示最佳比例尺
      that.map.setViewport(pointsArr)
      alert(
        '当前地图中心点：' +
          that.map.getCenter().getLng() +
          ',' +
          that.map.getCenter().getLat()
      )
    },

		// 添加多个点，并且点击弹出信息框
		pointsBtn () {
      const that = this
			const T = window.T
      const dataInfo = [
        [
          116.417854,
          39.921988,
          '地址：北京市东城区王府井大街88号乐天银泰百货八层'
        ],
        [116.406605, 39.921585, '地址：北京市东城区东华门大街'],
        [116.412222, 39.912345, '地址：北京市东城区正义路甲5号']
      ]

      for (let i = 0; i < dataInfo.length; i++) {
        const marker = new T.Marker(
          new T.LngLat(dataInfo[i][0], dataInfo[i][1])
        ) // 创建标注
        const content = dataInfo[i][2]
        that.map.addOverLay(marker) // 将标注添加到地图中
        that.addClickHandler(content, marker)
      }
    },
    addClickHandler (content, marker) {
			const that = this
			marker.addEventListener('click', function (e) {
				that.openInfo(content, e)
			})
		},
		openInfo (content, e) {
			const that = this
			const T = window.T
			const point = e.lnglat
			that.marker = new T.Marker(point) // 创建标注
			const markerInfoWin = new T.InfoWindow(content, {
				offset: new T.Point(0, -30)
			}) // 创建信息窗口对象
			that.map.openInfoWindow(markerInfoWin, point) // 开启信息窗口
		}
	},
	mounted () {
		const T = window.T
		const imageURL = 'http://t0.tianditu.gov.cn/img_w/wmts?' +
			'SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles' +
			'&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=7e56dbb209e1dc1c2935214c329ee774'
		const lay = new T.TileLayer(imageURL, { minZoom: 1, maxZoom: 16 })
		const config = { layers: [lay] }

		this.map = new T.Map('mapDiv', config) // 地图实例
		this.map.centerAndZoom(new T.LngLat(116.40769, 39.89945), 12)
		// 允许鼠标双击放大地图
		this.map.enableScrollWheelZoom()

		const ctrl = new T.Control.MapType([
			{
				title: '地图', // 地图控件上所要显示的图层名称
				icon: 'http://api.tianditu.gov.cn/v4.0/image/map/maptype/vector.png', // 地图控件上所要显示的图层图标（默认图标大小80x80）
				layer: window.TMAP_NORMAL_MAP // 地图类型对象，即MapType。
			},
			{
				title: '卫星混合',
				icon: 'http://api.tianditu.gov.cn/v4.0/image/map/maptype/satellitepoi.png',
				layer: window.TMAP_HYBRID_MAP
			}
		]) // 初始化地图类型选择控件
		this.map.addControl(ctrl) // 添加地图选择控件

		const control = new T.Control.Zoom({ position: window.T_ANCHOR_BOTTOM_RIGHT }) // 右下角
		this.map.addControl(control) // 添加缩放平移控件

		// this.map.setMapType(window.TMAP_HYBRID_MAP) // 设置地图位地星混合图层

		this.GetMaps()

		// this.onLoad()
	}
}
</script>

<style lang='less'>
@import '~ant-design-vue/lib/style/themes/default.less';
#mapDiv {
	position: absolute;
	width: 1200px;
	height: 100%;
}
</style>
