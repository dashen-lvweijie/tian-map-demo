<template>
	<view>
		<div id="mapDiv" style="position: absolute; width: 100%; height: 93%"></div>
		<div class="dkBtn" @click="test">打卡</div>
	</view>

</template>
<script module="vconsole" lang="renderjs">  
    import VConsole from 'vconsole'  
    new VConsole();  
</script>
<script>
	import proj4 from 'proj4';

	const WGS84 = '+proj=longlat +datum=WGS84 +no_defs';
	const CGCS2000 = '+proj=longlat +datum=CGCS2000 +no_defs';
	export default {
		name: "MapComponent",
		data() {
			return {
				map: null,
				zoom: 17,
				timer: null,
				user: {
					userLng: null,
					userLat: null
				},
				userMarker: null, // 用于保存用户位置的标记
			};
		},
		mounted() {
			this.loadScript().then(() => {
				this.initMap();

				this.timer = setInterval(() => {
					this.getLocation();
				}, 1000); // 每隔1秒执行一次
			}).catch((error) => {
				console.error("加载天地图脚本失败:", error);
			});
		},
		beforeDestroy() {
			clearInterval(this.timer);
		},
		methods: {
			// 打卡按钮
			test() {
				const userPosition = new T.LngLat(this.user.userLng, this.user.userLat);
				const centerPoint = new T.LngLat(117.0953, 31.833); // 打卡点
				const distance = this.map.getDistance(centerPoint, userPosition); // 计算距离
				if (distance <= 200) {
					uni.showToast({
						title:"打卡成功"
					})
					console.log("打卡成功");
				} else {
					uni.showToast({
						title:"未在打卡范围内"
					})
					console.log("未在打卡范围内");
				}
			},
			// 获取用户位置
			getLocation() {
				if (navigator.geolocation) {
					navigator.geolocation.getCurrentPosition((position) => {
						// 获取用户的 WGS84 坐标
						const userLng = position.coords.longitude;
						const userLat = position.coords.latitude;

						// 转换为 CGCS2000 坐标
						const cgcs2000Coordinates = proj4(WGS84, CGCS2000, [userLng, userLat]);

						this.user.userLng = cgcs2000Coordinates[0];
						this.user.userLat = cgcs2000Coordinates[1];

						const centerPoint = new T.LngLat(this.user.userLng, this.user.userLat);
						// 清除上一个用户位置的标记
						if (this.userMarker) {
							this.map.removeOverLay(this.userMarker);
						}
						// 创建新的用户位置标记
						this.userMarker = new T.Marker(centerPoint, {
							icon: new T.Icon({
								iconUrl: "https://api.map.baidu.com/images/marker_red.png",
								iconSize: new T.Point(25, 25),
								iconAnchor: new T.Point(13, 25)
							})
						});
						this.map.addOverLay(this.userMarker); // 将图钉标记添加到地图
					}, (error) => {
						console.error("获取位置失败:", error);
					});
				} else {
					console.log("浏览器不支持获取地理位置");
				}
			},

			// 加载脚本文件
			loadScript() {
				return new Promise((resolve, reject) => {
					const script = document.createElement("script");
					script.src =
						"https://api.tianditu.gov.cn/api?v=4.0&tk=2b2b83e9c356059677e06ec76036f05e";
					script.onload = resolve;
					script.onerror = reject;
					document.head.appendChild(script);
				});
			},
			// 初始化地图
			initMap() {
				this.map = new T.Map("mapDiv");
				const centerPoint = new T.LngLat(117.0953, 31.833);
				this.map.centerAndZoom(centerPoint, this.zoom);
				this.map.enableScrollWheelZoom();
				// 在中心点添加图钉标记和文本
				this.addMarkerWithLabel(centerPoint, "中安创谷");
			},
			// 添加目标点信息
			addMarkerWithLabel(position, text) {
				// 创建图钉标记
				const marker = new T.Marker(position);
				this.map.addOverLay(marker); // 将图钉标记添加到地图

				// 创建文本标签
				const label = new T.Label({
					text: text, // 文本内容
					position: position, // 文本位置，和标记位置相同
					offset: new T.Point(15, -30), // 设置偏移量，右上角显示
				});

				this.map.addOverLay(label); // 将文本标签添加到地图


				// 创建圆形范围
				const circle = new T.Circle(position, 200, {
					color: "blue",
					weight: 2,
					opacity: 0.5,
					fillColor: "orange",
					fillOpacity: 0.5,
				});

				this.map.addOverLay(circle); // 将圆形范围添加到地图

			},
		},
		watch: {
			zoom(newVal) {
				this.map.setZoom(newVal);
			},
		},
	};
</script>

<style scoped>
	/* 可以根据需要在这里添加样式 */
	* {
		padding: 0;
		margin: 0;
		box-sizing: border-box;
	}

	.dkBtn {
		position: absolute;
		bottom: 0;
		display: flex;
		justify-content: center;
		align-items: center;
		cursor: pointer;
		width: 100%;
		height: 7%;
		color: #fff;
		font-size: 20px;
		text-align: center;
		background-color: #007AFF;
	}
</style>