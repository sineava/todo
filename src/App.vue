<script>
export default {
	onLaunch: function() {
		// 判断是否缓存todo数据
		const todoData = uni.getStorageSync('todo')
		const repeatData = uni.getStorageSync('repeat')
		if (!todoData) {
			uni.setStorageSync('todo',[])
		} else {
			this.$scope.globalData.todo = todoData
		}
		if (!repeatData) {
			uni.setStorageSync('repeat',[])
		} else {
			this.$scope.globalData.repeat = repeatData
		}
	},
	onShow: function() {
		console.log('App Show');
	},
	onHide: function() {
		uni.setStorageSync('todo',getApp().globalData.todo)
		uni.setStorageSync('repeat',getApp().globalData.repeatData)
	},
	// 全局数据
	globalData: {  
		todo: [],
		repeatData: []
	}
};
</script>

<style>
/*每个页面公共css */
@import '@/static/colorui/main.css';
@import '@/static/colorui/icon.css';
@import '@/static/colorui/animation.css';
</style>
