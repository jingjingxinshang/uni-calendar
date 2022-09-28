<template>
	<view>
		<view>当前时间：{{currentYear}}年{{currentMonth}}月{{currentDay}}日</view>
		<br>
		<view @click="popupClick">{{currentDate?.year}}年{{currentDate?.month}}月-打开选择</view>
		<uni-popup ref="popup" type="center">
			<view style="width: 700rpx;height: 800rpx;background-color: #fff;">
				<picker-view indicator-style="height: 50px;" style="width: 100%; height: 600rpx;text-align: center;"
					:value="multiIndex" @change="pickerChange" class="picker-view">
					<picker-view-column>
						<view class="item" v-for="(item,index) in years" :key="index">
							<text v-if="isShowNong">{{item.nyear}}</text>
							<text v-else>{{item.year}}</text>
						</view>
					</picker-view-column>
					<picker-view-column>
						<view class="item" v-for="(item,index) in months" :key="index">
							<text v-if="isShowNong">{{item.nmonth}}</text>
							<text v-else>{{item.month}}</text>
						</view>
					</picker-view-column>
					<picker-view-column>
						<view class="item" v-for="(item,index) in days" :key="index">
							<text v-if="isShowNong">{{item.nday}}</text>
							<text v-else>{{item.day}}</text>
						</view>
					</picker-view-column>
				</picker-view>
				<view style="display: flex;justify-content: center;margin: 30rpx;">
					<checkbox-group @change="handleNong">
						<label>
							<checkbox :value="isShowNong" />农历
						</label>
					</checkbox-group>

				</view>
				<view style="display: flex;">
					<button style="flex: 1;" @click="popupCancel">取消</button>
					<button style="flex: 1;" @click="popupSubmit">确定</button>
				</view>

			</view>

		</uni-popup>
	</view>
</template>

<script setup>
	import {
		nextTick,
		onMounted,
		ref
	} from "vue";
	const popup = ref(null) // 选择框
	const years = ref([]) // 年数组，公历和农历共用
	const months = ref([]) // 月数组， 公历和农历共用
	const multiIndex = ref([0, 0, 0]) // 时间选择索引[1,4,11] =》 1992，5，12 对应为3列索引
	const currentDate = ref(null) // 选择时间
	const currentYear = ref(null) // 当前年
	const currentMonth = ref(null) // 当前月
	const currentDay = ref(null)  // 当前日
	const nowYear = ref(null) // 日期选择器选择年
	const nowMonth = ref(1) // 日期选择其选择月
	const days = ref([]) // 日数组，公农历共用
	const isShowNong = ref(false) // 是否展示农历 ，默认不展示，
	const emit = defineEmits(['popupSubmit']) // emit 确定按钮
	onMounted(() => {
		// 获取当前时间 因为没有正式数据，模拟 1992 5 12
		currentYear.value = 1922 || new Date().getFullYear()
		currentMonth.value = 5 || new Date().getMonth()
		currentDay.value = 12 || new Date().getDay()
		// 展示选择时间
		currentDate.value = {
			year: currentYear.value,
			month: currentMonth.value,
			day: currentDay.value
		}
	
		console.log('ref', popup)
		// 获取年 月 日数组，参数为true 是为了获取连续获取
		getYears(true)
	})
	// 打开日期选择
	const popupClick = () => {
		popup.value.open()
	}
	// 关闭日期选择
	const popupCancel = () => {
		popup.value.close()
	}
	// 日期选择提交
	const popupSubmit = () => {
		console.log(multiIndex)
		// 根据 选择器绑定索引数组 获取选择年月日
		const data = {
			year: years.value[multiIndex.value[0]].year,
			month: months.value[multiIndex.value[1]].month,
			day: days.value[multiIndex.value[2]].day,
		}
		currentDate.value = {
			...data
		}
		popup.value.close()
		// 将数据传给父组件
		emit('popupSubmit', multiIndex.value, data)
	}
	// 获取年数组
	const getYears = (flag) => {
		uni.request({
			url: 'https://www.aiyccc.com/rili_year',
			method: 'POST',
			success: (res) => {
				if (res.data.data.length > 0) {
					years.value = res.data.data
					// 根据参数判断是否继续向下获取月 
					if (flag) {
						getMonths(currentYear.value || years.value[0].year, true)
					}
				} else {
					years.value = []
				}
			}
		});
	}
	// 获取公历月数组
	const getMonths = (year, flag) => {
		uni.request({
			url: 'https://www.aiyccc.com/rili_month',
			method: 'POST',
			data: {
				year: year
			},
			success: (res) => {
				if (res.data.data.length > 0) {
					months.value = res.data.data
					// 根据参数判断是否继续向下获取日 
					if (flag) {
						getDays(currentYear.value ||years.value[0].year, currentMonth.value ||months.value[0].month, flag)
					}

				} else {
					months.value = []
				}

			}
		});
	}
	// 获取农历月数组
	const getnMonths = (year) => {
		uni.request({
			url: 'https://www.aiyccc.com/rili_nmonth',
			method: 'POST',
			data: {
				year: year
			},
			success: (res) => {
				if (res.data.data.length > 0) {
					months.value = res.data.data
				} else {
					months.value = []
				}

			}
		});
	}
	// 获取日数组
	const getDays = (year, month, flag) => {
		new Promise((resolve, reject) => {
			uni.request({
				url: 'https://www.aiyccc.com/rili_day',
				method: 'POST',
				data: {
					year: year,
					month: month
				},
				success: (res) => {
					if (res.data.data.length > 0) {
						console.log(res.data);
						days.value = res.data.data
						// 根据参数判断是否继续向下获取日 
						if (flag) {
							// 根据当前日期自动锚点。仅作为初始化数据时使用
							nextTick( () => {
								const currentYearIndex = years.value.findIndex(item => item.year === currentYear.value)
								const currentMonthIndex = months.value.findIndex(item => item.month === currentMonth.value) 
								const currentDayIndex = days.value.findIndex(item => item.day === currentDay.value) 
								multiIndex.value = [currentYearIndex, currentMonthIndex, currentDayIndex]
								console.log(multiIndex.value)
							})
							
						}
					} else {
						days.value = []
					}
					resolve(res)
				}
			});
		})

	}
	// 获取公历日
	const getnDays = (year, month) => {
		uni.request({
			url: 'https://www.aiyccc.com/rili_nday',
			method: 'POST',
			data: {
				year: year,
				month: month
			},
			success: (res) => {
				if (res.data.data.length > 0) {
					console.log(res.data);
					days.value = res.data.data
				} else {
					days.value = []
				}
			}
		});
	}
	// 农历方法
	const handleNong = async () => {
		// 公农历互转接口可以不调用，通过选择的日拿到相对应的数据
		isShowNong.value = !isShowNong.value
		// 展示农历
		if (isShowNong.value) {
			console.log(true, years.value, multiIndex.value)
			// 获取公历数据
			const year = years.value[multiIndex.value[0]].year
			const month = months.value[multiIndex.value[1]].month
			const day = days.value[multiIndex.value[2]].day
			const p = {
				type: 1,
				year: year,
				month: month,
				day: day
			}
			uni.request({
				url: 'https://www.aiyccc.com/rili_transformation',
				data: {
					...p
				},
				success: async (res) => {
					const {
						nyear,
						nmonth,
						nday
					} = res.data.data
					// 根据公历转换为农历的数据 获取农历月数组
					getnMonths(years.value.find(item => item.nyear === nyear).nyear)
					// 获取农历日数据
					uni.request({
						url: 'https://www.aiyccc.com/rili_nday',
						method: 'POST',
						data: {
							year: years.value.find(item => item.nyear === nyear).nyear,
							month: months.value.find(item => item.nmonth === nmonth).nmonth
						},
						success: (res) => {
							if (res.data.data.length > 0) {
								console.log(res.data);
								days.value = res.data.data
								// 农历数据索引
								const lYearIndex = years.value.findIndex(item => item
									.nyear === nyear)
								const lMonthIndex = months.value.findIndex(item => item
									.nmonth ===
									nmonth)
								const lDayIndex = days.value.findIndex(item => item
									.nday === nday)
								// 索引数据
								multiIndex.value = [lYearIndex, lMonthIndex, lDayIndex]
								console.log(multiIndex.value)
							} else {
								days.value = []
							}
						}
					});

				}
			})
		} else {
			// 农历数据
			const nyear = years.value[multiIndex.value[0]].nyear
			const nmonth = months.value[multiIndex.value[1]].nmonth
			const nday = days.value[multiIndex.value[2]].nday
			const p = {
				type: 2,
				year: nyear,
				month: nmonth,
				day: nday
			}
			uni.request({
				url: 'https://www.aiyccc.com/rili_transformation',
				data: {
					...p
				},
				success: async (res) => {
					const {
						year,
						month,
						day
					} = res.data.data
					// 根据转换的值获取公历月
					getMonths(years.value.find(item => item.year === year).year)
					// 获取公历日数据
					uni.request({
						url: 'https://www.aiyccc.com/rili_day',
						method: 'POST',
						data: {
							year: years.value.find(item => item.year === year).year,
							month: months.value.find(item => item.month === month).month
						},
						success: (res) => {
							if (res.data.data.length > 0) {
								console.log(res.data);
								days.value = res.data.data
								// 处理索引
								const lYearIndex = years.value.findIndex(item => item
									.year === year)
								const lMonthIndex = months.value.findIndex(item => item
									.month === month)
								const lDayIndex = days.value.findIndex(item => item.day ===
									day)
								multiIndex.value = [lYearIndex, lMonthIndex, lDayIndex]
								console.log(multiIndex.value)
							} else {
								days.value = []
							}
						}
					});



				}
			})
		}
	}
	// 日期选择方法
	const pickerChange = (e) => {
		console.log('我执行了', e)
		// 选择的索引
		const val = e.detail.value
		const year = val[0]
		const month = val[1]
		const day = val[2]
		// 根据索引获取 年月的数据
		nowYear.value = years.value[year]
		nowMonth.value = months.value[month]
		// 判断年索引是否变化，变化之后获取新的月日数据
		if (multiIndex.value[0] !== year) {
			// 判断是否是农历
			if (isShowNong.value) {
				console.log(nowYear.value.nyear, true, nowMonth.value.nmonth)
				getnMonths(nowYear.value.nyear)
				getnDays(nowYear.value.nyear, nowMonth.value.nmonth)
			} else {
				console.log(1)
				getMonths(nowYear.value.year)
				getDays(nowYear.value.year, nowMonth.value.month)
			}

		}
		if (multiIndex.value[1] !== month) {
			if (isShowNong.value) {
				console.log(nowYear.value.nyear, false, nowMonth.value.nmonth)
				getnDays(nowYear.value.nyear, nowMonth.value.nmonth)
			} else {
				getDays(nowYear.value.year, nowMonth.value.month)
			}
		}
		multiIndex.value = val

	}
</script>

<style>
	.item {
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>
