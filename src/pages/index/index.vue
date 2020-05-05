<template>
	<view class="content">
		<dark-calendar
			:signeddates="signeddates"
			@selectDate="selectDate"
		></dark-calendar>
		<view class="card-list">
			<view @touchstart="start" @touchend="end($event,item)"
				:class="{ item, complete: item.complete }"
				v-for="(item, index) in cards"
				:key="index"
			>
				<view class="time">{{ item.datetime.time }}</view>
				<view class="content">{{ item.content }}</view>
				<view class="btn-wrapper">
					<checkbox :checked="item.complete" @click="change(index)" />
				</view>
				<view
					class="bar"
					:style="{
						backgroundColor: item.bgcolor ? item.bgcolor : 'green'
					}"
				></view>
			</view>
		</view>

		<!-- 悬浮按钮 -->
		<view class="addbtn" @tap="changeModal">+</view>

		<!-- model -->
		<view class="cu-modal" :class="{ show: show }">
			<view class="cu-dialog">
				<view class="cu-bar bg-white justify-end">
					<view class="content">选项卡</view>
				</view>
				<view class="padding-xl">
					<view class="cu-form-group">
						<view class="title">代办内容</view>
						<input class="content" v-model="content" />
					</view>
					<view class="cu-form-group">
						<view class="title">每日重复</view>
						<checkbox
							:checked="repeat"
							@click="this.repeat = !this.repeat"
						></checkbox>
					</view>
					<view class="cu-form-group">
						<view class="title">日期选择</view>
						<picker
							mode="date"
							:value="date"
							start="1970-01-01"
							end="2300-01-01"
							@change="dateChange"
						>
							<view class="picker">{{ date }}</view>
						</picker>
					</view>
					<view class="cu-form-group">
						<view class="title">时间选择</view>
						<picker
							mode="time"
							:value="time"
							start="00:00"
							end="23:59"
							@change="timeChange"
						>
							<view class="picker">{{ time }}</view>
						</picker>
					</view>
					<view class="cu-form-group">
						<button class="cu-btn round bg-red" @click="bgcolor = '#e54d42'">嫣红</button>
						<button class="cu-btn round bg-olive" @click="bgcolor = '#8dc63f'">橄榄</button>
						<button class="cu-btn round bg-pink" @click="bgcolor = '#e03997'">桃粉</button>
						<button class="cu-btn round bg-cyan" @click="bgcolor = '#1cbbb4'">天青</button>
					</view>
				</view>
				<view class="cu-bar bg-white justify-end">
					<view class="action">
						<button
							class="cu-btn line-green text-green"
							@tap="changeModal"
						>
							取消
						</button>
						<button class="cu-btn bg-green margin-left" @tap="sure">
							确定
						</button>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			title: 'Hello',
			signeddates: ['2020-05-02', '2020-05-03'],
			cards: [],
			flag: false,
			modalName: null,
			show: false,
			date: '',
			time: '12:02',
			content: '',
			repeat: false, // 是否重复
			repeatData: getApp().globalData.repeatData,
			bgcolor: '#0864ef',
			position: {
				clientX: 0
			}
		}
	},
	onLoad() {
		const date = new Date()
		this.selectDate(
			`${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()}`
		)
		this.date = `${date.getFullYear()}-${
			date.getMonth() + 1 < 10
				? '0' + (date.getMonth() + 1)
				: date.getMonth() + 1
		}-${date.getDate() < 10 ? '0' + date.getDate() : date.getDate()}`
	},
	onHide() {
		getApp().globalData.repeatData = this.repeatData
	},
	methods: {
		async selectDate(date) {
			// 重复数据获取
			let repeatDate = null
			if (this.repeatData) {
				const dateMill = Date.parse(new Date(date))
				repeatDate = this.repeatData.filter(time => {
					return Date.parse(new Date(time.datetime.date)) < dateMill
				})
			}
			const data = getApp().globalData.todo
			this.cards = data
				.filter(item => item.datetime.date == date)
				.concat(repeatDate)
		},
		change(index) {
			this.cards[index].complete = !this.cards[index].complete
		},
		changeModal() {
			this.show = !this.show
		},
		dateChange(date) {
			this.date = date.target.value
		},
		timeChange(time) {
			this.time = time.target.value
		},
		getCurrent() {
			const date = new Date()
			this.date = `${date.getFullYear()}-${date.getMonth() +
				1}-${date.getDate()}`
		},
		sure() {
			if (this.content.trim()) {
				const $global = getApp().globalData.todo
				const data = {
					datetime: {
						date: this.date.replace(/-0/g, '-'),
						time: this.time
					},
					content: this.content,
					repeat: this.repeat,
					complete: false,
					bgcolor: this.bgcolor
				}
				if (this.repeat) {
					this.repeatData.push(data)
				}
				$global.push(data)
				const date = new Date()
				this.selectDate(
					`${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()}`
				)
			}
			// 隐藏模态框
			this.changeModal()
		},
		start(e) {
			this.position.clientX = e.changedTouches[0].clientX
		},
		end(e,item) {
			const subX = e.changedTouches[0].clientX - this.position.clientX
			if (subX < -100) {
				if (item.repeat) {
					this.repeatData = this.repeatData.filter(ob => ob != item)
				}
				const $global = getApp().globalData.todo
				this.cards = this.cards.filter(ob => ob != item)
				getApp().globalData.todo = $global.filter(ob => ob != item)
			}
		}
	}
}
</script>

<style lang="less">
.content {
	width: 750rpx;
	text-align: center;
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
			transition: all 2s ease;
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
	.addbtn {
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

	.cu-modal {
		.cu-dialog {
			.padding-xl {
				background-color: #fff;
				.cu-form-group {
					border: none;
					.title {
						font-size: 22rpx;
					}
					.content {
						text-align: right;
					}
				}
			}
		}
	}
}

// 颜色
.cu-btn.round:hover {
	box-shadow: 0px 0px 10px rgba(0,0,0,.5);
}
</style>
