<template>
	<movable-area class="drag-sort" :style="{height: currentListLength * height + 'px'}" id="drag">
		<movable-view v-for="(item, index) in currentList" :key="index" :x="0" :y="item.y" direction="vertical" disabled
		 damping="40" :animation="active !== index" class="drag-sort-item" style="height:110px" :class="{'active': active === index, 'vh-1px-t': item.index > 0}">
			<view class="item">
				<view class="image-text-list space-between">
					<view class="image" v-if="item.image.length > 0">
						<image :src="common.imgUrl(item.image) " @tap="previewImage(index)" mode="aspectFill"></image>
						<view class="delete" @tap="Remove(index)">x</view>
					</view>
					<view class="uni-uploader__input-box" v-if="item.image.length == 0" @tap="chooseImage(index)">
						<view class="uni-uploader__input" ></view>
						<view class="input_cell">
							<image :src="common.imgUrl('/static/images/add.png')"></image>
							<view>选择图片</view>
						</view>
					</view>
					<view class="text">
						<textarea placeholder="添加文字" :maxlength="-1" :value="item.content" :data-index="index" @input="contentInput" />
						</view>
				</view>
			</view>
			<view class="touch-tight" v-if="item.image.length > 0">
				<view class="ico_drag"></view>
			</view>
		</movable-view>
		<movable-view class="touch" :x="2000" @touchstart="touchstart" @touchmove="touchmove" @touchend="touchend"
		 catchtouchstart catchtouchmove catchtouchend></movable-view>
	</movable-area>
</template>

<script>
	export default {
		name: 'drag-sort',
		mixins: [],
		components: {},
		data() {
			return {
				height: 110, // 高度
				currentList: [],
				active: -1, // 当前激活的item
				itemIndex: 0, // 当前激活的item的原index
				topY: 0, // 距离顶部的距离
				deviationY: 0 // 偏移量
			}
		},
		computed: {
			currentListLength() {
				return this.currentList.length 
			}
		},
		props: {
			list: {
				type: Array,
				default: () => {
					return []
				}
			},
			props: {
				type: Object,
				default: () => {
					return {
						label: 'label',
						value: 'value'
					}
				}
			}
		},
		watch: {
			list(val) {
				this.onUpdateCurrentList()
			}
		},
		created() {
			this.onUpdateCurrentList()
		},
		mounted() {},
		updated() {},
		filters: {},
		methods: {
			contentInput(e){
				this.$emit("contentInput",{
					index: e.currentTarget.dataset.index,
					content: e.detail.value
				})
			},
			chooseImage(index){
				this.$emit("chooseImage",index)
			},
			Remove(index){
				this.$emit("Remove",index)
			},
			previewImage(index){
				this.$emit("previewImage",index)
			},
			onUpdateCurrentList() {
				let arr = []
				for (const key in this.list) {
					arr.push({
						...this.list[key],
						index: Number(key),
						y: key * this.height,
						animation: true
					})
				}
				this.currentList = arr
				
			},
			touchstart(e) {
				// 计算y轴点击位置
				var query = wx.createSelectorQuery().in(this)
				query.select('#drag').boundingClientRect()
				query.exec((res) => {
					this.topY = res[0].top
					let touchY = e.mp.touches[0].clientY - res[0].top
					this.deviationY = touchY % this.height
					// console.log(touchY)
					for (const key in this.currentList) {
						if ((this.currentList[key].index * this.height < touchY) && ((this.currentList[key].index + 1) * this.height >
								touchY)) {
							this.active = Number(key)
							this.itemIndex = this.currentList[key].index
							break
						}
					}
				})
			},
			touchmove(e) {
				if (this.active < 0) return
				let touchY = (e.mp.touches[0].clientY - this.topY) - this.deviationY
				// console.log(touchY)
				this.currentList[this.active].y = touchY
				for (const key in this.currentList) {
					// 跳过当前操作的item
					if (this.currentList[key].index !== this.currentList[this.active].index) {
						if (this.currentList[key].index > this.currentList[this.active].index) {
							if (touchY > this.currentList[key].index * this.height - this.height / 2) {
								this.currentList[this.active].index = this.currentList[key].index
								this.currentList[key].index = this.currentList[key].index - 1
								this.currentList[key].y = this.currentList[key].index * this.height
								break
							}
						} else {
							if (touchY < this.currentList[key].index * this.height + this.height / 2) {
								this.currentList[this.active].index = this.currentList[key].index
								this.currentList[key].index = this.currentList[key].index + 1
								this.currentList[key].y = this.currentList[key].index * this.height
								break
							}
						}
					}
				}
			},
			touchend(e) {
				if ((this.itemIndex !== this.currentList[this.active].index) && (this.active > -1)) {
					this.$emit('change', {
						// 操作值
						data: (() => {
							let data = {
								...this.currentList[this.active]
							}
							delete data.index
							delete data.y
							delete data.animation
							return data
						})(),
						// 插队的位置前面的值
						frontData: (() => {
							for (const iterator of this.currentList) {
								if (this.currentList[this.active].index - 1 === iterator.index) {
									let data = {
										...iterator
									}
									delete data.index
									delete data.y
									delete data.animation
									return data
								}
							}
						})()
					})
				}
				this.currentList[this.active].animation = true
				this.currentList[this.active].y = this.currentList[this.active].index * this.height
				this.active = -1
			}
		}
	}
</script>

<style lang='less' scoped>
	@import "~./1px.less";
	.drag-sort {
		width: 100%;
	}
.image-text-list {
		padding: 15px 0;
		
	}
	.image {
		width: 160upx;
		height: 160upx;
		position: relative;
	}
	.image>image {
		width: 100%;
		height: 100%;
	}
	.text {
		height: 160upx;
		width: 73%;
	}
	.text>textarea {
		height: 100%;
		/* border:1px solid #000; */
		width: 100%;
	}
	.add {
		padding: 15px 0;
	}
	.uni-uploader__input-box {
		margin: 0;
		width: 160upx;
		height: 160upx;
		position: relative;
		background-color:#f4f5f6;
		border-radius: 10rpx;
		overflow:hidden;
	}
	.input_cell{
		position: absolute;
		top: 50%;
		left: 50%;
		width: 104rpx;
		transform: translate(-50%,-50%);
		font-size:26rpx;
		color:#606266;
		display: flex;
		flex-direction:column;
	}
	.input_cell>image{
		width: 40rpx;
		height: 40rpx;
		margin-left: 32rpx;
		margin-bottom: 20rpx;
	}
	.uni-uploader__input {
		width: 100%;
		height: 100%;
		border: 1px solid #f4f5f6 !important;
	}
	.delete {
		width: 40upx;
		height: 40upx;
		border-radius: 50%;
		text-align: center;
		line-height: 32upx;
		position: absolute;
		right: -15upx;
		top: -15upx;
		color: #fff;
		background-color: #ff6638;
		font-size: 32upx;
	}
	.drag-sort-item {
		position: absolute !important;
		display: flex;
		align-items: center;
		width: 100%;
		padding: 0;
		margin: 0;
		background: #fff;
		box-sizing: border-box;
		border-bottom: 1px solid #f7f7f7;
		.item {
			flex: 1;
		}
		.touch-tight {
			width: 24px;
			display: flex;
			justify-content: center;
		}
	}
	.touch {
		height: 100%;
		width: 50px;
	}
	.ico_drag {
		display: inline-block;
		width: 24px;
		height: 12px;
		background: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADAAAAAYCAYAAAC8/X7cAAAAAXNSR0IArs4c6QAAAEtJREFUWAnt1cEJACAMA0B1/506moIr5FEK51+Jl0d2Vd01+JzB2X90H5jeoPwECBDIBLYlzgDj25Y4JvQAAQIERgtY4u76LHF3Aw8rGQnK3sYAXQAAAABJRU5ErkJggg==) 0 0 no-repeat;
		background-size: 100% auto;
	}
	.active {
		box-shadow: 0 0 40rpx #DDDDDD;
		z-index: 99;
	}
</style>