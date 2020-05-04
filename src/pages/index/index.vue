<template>
	<view class="content">
		<dark-calendar :signeddates="signeddates" @selectDate="selectDate"></dark-calendar>
		<view class="card-list">
			<view :class="{item,complete:item.complete}" v-for="(item,index) in cards" :key="index">
				<view class="time">{{item.datetime.time}}</view>
				<view class="content">{{item.content}}</view>
				<view class="btn-wrapper">
					<checkbox :checked="item.complete" @click="change(index)"/>
				</view>
				<view class="bar" :style="{backgroundColor:item.bgcolor ? item.bgcolor : 'green'}"></view>
			</view>
		</view>
		
		<!-- 悬浮按钮 -->
		<view class="add" hover-class="hover">+</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				signeddates: [
					'2020-05-02',
					'2020-05-03'
				],
				cards: [],
				flag: false
			}
		},
		onLoad() {
			const date = new Date()
			this.selectDate(`${date.getFullYear()}-${date.getMonth()+1}-${date.getDate()}`)
		},
		methods: {
			async selectDate (date) {
				console.log(1);
				const data = await uni.request({
					url: `http://192.168.1.4:3000/todos?datetime.date=${date}`
				}).then(data => data[1].data)
				this.cards = data
			},
			change (index) {
				this.cards[index].complete = !this.cards[index].complete
			}
		}
	}
</script>

<style lang="less">
.content {
	width: 750rpx;
	text-align: center;
	// background-color: #f4f4f4;
	.card-list {
		width: 700rpx;
		margin: 0 auto;
		.item {
			padding: 30rpx 30rpx 30rpx 0;
			margin-top: 30rpx;
			border-radius: 5px;
			background-color: #fff;
			display: flex;
			align-items: center;
			font-size: 14px;
			position: relative;
			.time {
				color: #bababa;
				margin-left: 30rpx;
			}
			.content {
				background-color: #fff;
			}
			&.complete {
				.time {
					text-decoration: line-through;
				}
				.content {
					text-decoration: line-through;
				}
			}
			.bar {
				position: absolute;
				width: 10rpx;
				height: 60%;
				left: 0;
				border-radius: 10rpx;
			}
		}
	}
	.add {
		position: fixed;
		bottom: 20rpx;
		right: 20rpx;
		width: 80rpx;
		height: 80rpx;
		background-color: red;
		border-radius: 50%;
		display: flex;
		justify-content: center;
		align-items: center;
		color: white;
		font-size: 50rpx;
		font-weight: bold;
		transition: transform ease 1s;
		&:hover {
			transform: rotate(45deg);
		}
	}
}
</style>
