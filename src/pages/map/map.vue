<template>
	<view class="map">
		<view class="map-item">
			<map
				style="width: 100%; height: 400rpx;"
				v-if="latitude"
				:latitude="latitude"
				:longitude="longitude"
				:show-location="show"
				:markers="markers"
				:polyline="polyline"
				enable-overlooking
			></map>
		</view>
		<view class="position">
			<view class="cu-form-group">
				<view class="title">出发地</view>
				<input
					class="content"
					v-model="startPosition"
					@focus="ck('start')"
				/>
			</view>
			<view class="cu-form-group dest">
				<view class="title">目的地</view>
				<input
					class="content"
					v-model="endPosition"
					@focus="ck('end')"
				/>
			</view>
			<image
				class="go"
				src="../../static/image/go.png"
				@click="go"
			></image>
		</view>

		<!-- 联想 -->
		<view class="position_keywords">
			<view
				class="position_item"
				v-for="item in positionData"
				:data-item="item"
				:key="index"
				@click="surePosition"
			>
				<image
					src="../../static/image/location.png"
					class="location_icon"
				></image>
				{{ item.address }}
			</view>
		</view>
	</view>
</template>

<script>
const QQMapWX = require('../../static/sdk/qqmap-wx-jssdk.min.js')
var qqmapsdk = new QQMapWX({
	key: 'GNKBZ-2RUCP-RO3DE-LMN5S-IEKX6-CMFT3' // 必填
})
export default {
	data() {
		return {
			title: 'map',
			latitude: '',
			longitude: '',
			endlatitude: '',
			endlongitude: '',
			covers: [],
			startPosition: '',
			endPosition: '',
			positionData: [],
			cursor: 'start',
			markers: [],
			polyline: [], // 路线规划
			show: true
		}
	},
	onShow() {
		const $that = this
		uni.getLocation({
			type: 'gcj02',
			success: function(res) {
				$that.latitude = res.latitude
				$that.longitude = res.longitude
				$that.covers.push({
					latitude: res.latitude,
					longitude: res.longitude,
					iconPath: '../../static/image/location.png'
				})
			}
		})
	},
	methods: {
		getPosition(keyword) {
			if (!keyword) return
			var _this = this
			qqmapsdk.getSuggestion({
				keyword: keyword,
				policy: 1,
				region: '重庆',
				success: function(res) {
					_this.positionData = res.data
				}
			})
		},
		ck(el) {
			// 用于辨认定位于开始还是结束位置
			this.cursor = el
		},
		surePosition(data) {
			const item = data.target.dataset.item
			this.cursor === 'start'
				? (this.startPosition = item.address)
				: (this.endPosition = item.address)
			if (this.cursor === 'start') {
				this.longitude = item.location.lng
				this.latitude = item.location.lat
				this.markers = [
					{
						latitude: item.location.lat,
						longitude: item.location.lng,
						iconPath: '../../static/image/location.png'
					}
				]
			}
			if (this.cursor === 'end') {
				this.endlongitude = item.location.lng
				this.endlatitude = item.location.lat
				this.markers.length > 1
					? this.markers.pop().push({
							latitude: item.location.lat,
							longitude: item.location.lng,
							iconPath: '../../static/image/desition.png'
					  })
					: this.markers.push({
							latitude: item.location.lat,
							longitude: item.location.lng,
							iconPath: '../../static/image/desition.png'
					  })
			}
		},
		go() {
			var _this = this
			//调用距离计算接口
			qqmapsdk.direction({
				mode: 'walking',
				from: this.latitude + ',' + this.longitude,
				to: this.endlatitude + ',' + this.endlongitude,
				success: function(res) {
					var ret = res
					var coors = ret.result.routes[0].polyline,
						pl = []
					var kr = 1000000
					for (var i = 2; i < coors.length; i++) {
						coors[i] = Number(coors[i - 2]) + Number(coors[i]) / kr
					}
					for (var i = 0; i < coors.length; i += 2) {
						pl.push({ latitude: coors[i], longitude: coors[i + 1] })
					}
					_this.polyline = [
						{
							points: pl,
							color: '#3498db',
							width: 4
						}
					]
				}
			})
			// this.show = false
			this.positionData = []
		}
	},
	watch: {
		startPosition(keyword) {
			this.getPosition(keyword)
		},
		endPosition(keyword) {
			this.getPosition(keyword)
		}
	}
}
</script>

<style lang="less">
.map-item {
	width: 100%;
	height: 400rpx;
	box-sizing: border-box;
}
.position_keywords {
	// position: absolute;
	// top: 0;
	// left: 20px;
}
.position_item {
	width: 100%;
	margin: 10rpx;
}

.location_icon {
	width: 20rpx;
	height: 20rpx;
}

.position {
	position: relative;
	.go {
		display: block;
		width: 80rpx;
		height: 80rpx;
		position: absolute;
		right: 40rpx;
		top: 50%;
		transform: translateY(-50%);
	}
}

.cu-form-group+.cu-form-group {
	border-top: none;
}

.dest {
	position: relative;
	&::before {
		position: absolute;
		content: '';
		width: 80%;
		left: 0;
		top: 0;
		height: 1rpx;
		background-color: #eee;
	}
}
</style>
