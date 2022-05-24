<template>
	<!-- <div id="mapDiv" style="position:absolute;width:100%; height:100%"></div> -->
	<div id="mapDiv"></div>
</template>

<script>
// import { map } from 'store/storages/all'
const TMAP_NORMAL_MAP = 'TMAP_NORMAL_MAP' // 普通街道视图
// const TMAP_SATELLITE_MAP = 'TMAP_SATELLITE_MAP' // 卫星视图
const TMAP_HYBRID_MAP = 'TMAP_HYBRID_MAP' // 卫星和路网的混合视图
// const TMAP_TERRAIN_MAP = 'TMAP_TERRAIN_MAP' // 地形视图
// const TMAP_TERRAIN_HYBRID_MAP = 'TMAP_TERRAIN_HYBRID_MAP' // 地形和路网的混合视图

export default {
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
