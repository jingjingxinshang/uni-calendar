<template>
	<view class="">
    <view @click="openPopup">
      
    </view>
		<calendars @popupSubmit="popupSubmit"></calendars>
		<view class="">
			选择时间为：{{currentDate}}
			<!-- <picker-view indicator-style="height: 50px;" style="width: 100%; height: 300px;" :value="multiIndex" @change="pickerChange"
				class="picker-view">
				<picker-view-column>
					<view class="item" v-for="(item,index) in array" :key="index">{{item.name}}({{item.nongName}})</view>
				</picker-view-column>
				<picker-view-column>
					<view class="item" v-for="(item,index) in array[multiIndex[0]]?.children" :key="index">{{item.name}}({{item.nongName}})</view>
				</picker-view-column>
				<picker-view-column>
					<view class="item" v-for="(item,index) in array[multiIndex[0]]?.children[multiIndex[1]]?.children" :key="index">{{item.name}}({{item.nongName}})</view>
				</picker-view-column>
			</picker-view> -->
		</view>
		
	</view>
</template>

<script>
	import calendar from './gong2nong.js'
	import calendars from './calendar.vue'
	export default {
		components: {
			calendars
		},
		data() {
			return {
				title: 'Hello',
				multiIndex: [0,0,0],
				array: [],
				multiArrayYear: [],
				multiArrayMouth: [],
				multiArrayDay: [],
				currentDate: ''
			}
		},
		onLoad() {

		},
		onShow() {
			// this.initDate()
			// this.initMultiArray()
			// console.log(calendar.solar2lunar(2020))
		},
		computed: {
			multiArray() {
				return this.array
			}
		},
		methods: {
			popupSubmit(arrayIndex,data) {
				this.multiIndex = [...arrayIndex]
				this.currentDate = data.year + '年' + data.month + '月' + data.day + '日'
			},
			pickerChange(e) {
				console.log('触发', e, this.multiIndex)
				const val = e.detail.value
				this.multiIndex = val
				console.log(val)
			},
			initMultiArray() {
				this.multiArrayYear = this.array
				this.multiArrayMouth = this.multiArrayYear[0].children
				this.multiArrayDay = this.multiArrayMouth[0].children
				console.log(this.multiArrayDay, this.multiArrayMouth, this.multiArrayYear)
				// const one = []
				// for (const a of this.array) {
				// 	one.push(a.name)
				// }
				// this.multiArray[0] =
			},
			initDate() {
				const date = new Date()
				const years = []
				for (let year = 2018; year <= date.getFullYear() + 10; year++) { // 获取从2018年到当前年份 + 10
					const mouthArray = []
					for (let mouth = 1; mouth <= 12; mouth++) {
						const days = this.getMouthDay(year, mouth) // 根据平闰年 获取天数
						const dayArray = []
						for (let day = 1; day <= days; day++) {
							dayArray.push({
								id: parseInt(year + '' + (mouth < 10 ? '0' + mouth : mouth) + '' + (day < 10 ?
									'0' + day : day)),
								name: day + '日',
								nongName: calendar.solar2lunar(year, mouth, day).IDayCn
							})
						}
						mouthArray.push({
							id: parseInt(year + '' + (mouth < 10 ? '0' + mouth : mouth)),
							name: mouth + '月',
							nongName: calendar.solar2lunar(year, mouth,1).IMonthCn,
							children: dayArray
						})
					}
					years.push({
						id: year,
						name: year + '年',
						nongName: calendar.solar2lunar(year, 1, 1).lYear,
						children: mouthArray
					})
				}
				console.log(years)
				this.array = [...years]
			},
			// 平闰年 每月天数获取
			getMouthDay(year, month) {
				var bigMonth = [1, 3, 5, 7, 8, 10, 12];
				var day = 0;
				if (!year || !month) return day;
				if (month === 2) {
					if (year % 100 === 0) {
						// 整百
						year % 400 === 0 ? day = 29 : day = 28
					} else {
						// 非整百
						year % 4 === 0 ? day = 29 : day = 28
					}
				} else {
					// 判断是否是大月份
					if (bigMonth.indexOf(month) === -1) {
						day = 30
					} else {
						day = 31
					}
				}
				return day
			},
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>
