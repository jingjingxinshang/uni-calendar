<template>
	<view>
		<view @click="popupClick">打开</view>
		<uni-popup ref="popup" type="center">
			<view style="width: 700rpx;height: 600rpx;background-color: azure;">
				<picker-view indicator-style="height: 50px;" style="width: 100%; height: 600rpx;text-align: center;"
					:value="multiIndex" @change="pickerChange" class="picker-view">
					<picker-view-column>
						<view class="item" v-for="(item,index) in years" :key="index">
							<text>{{item.nyear}}</text>
							<text>-{{item.year}}</text>
							<!-- <text v-if="isShowNong">{{item.nyear}}</text>
									<text v-else>{{item.year}}</text> -->
						</view>
					</picker-view-column>
					<picker-view-column>
						<view class="item" v-for="(item,index) in months" :key="index">
							<text>{{item.nmonth}}</text>
							<text>-{{item.month}}</text>
							<!-- <text v-if="isShowNong">{{item.nmonth}}</text>
									<text v-else>{{item.month}}</text> -->
						</view>
					</picker-view-column>
					<picker-view-column>
						<view class="item" v-for="(item,index) in days" :key="index">
							<text>{{item.nday}}</text>
							<text>-{{item.day}}</text>
							<!-- 	<text v-if="isShowNong">{{item.nday}}</text>
										<text v-else>{{item.day}}</text> -->
						</view>
					</picker-view-column>
				</picker-view>
				<view class="">
					<button @click="handleNong">农历</button>
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
	import calendar from './gong2nong.js'
	import {
		onMounted,
		ref
	} from "vue";
	const popup = ref(null)
	const years = ref([])
	const months = ref([])
	const multiIndex = ref([0, 0, 0])
	const currentYear = ref(null)
	const currentMonth = ref(null)
	const nowYear = ref(null)
	const nowMonth = ref(1)
	const days = ref([])
	const isShowNong = ref(false)
	const emit = defineEmits(['popupSubmit','popupClick'])
	onMounted(() => {
		currentYear.value = new Date().getFullYear()
		currentMonth.value = new Date().getMonth()
		// nowYear.value = currentYear.value
		// nowMonth.value = currentMonth.value
		console.log('ref', popup)
		getYears()
	})
	const popupClick = () => {
		popup.value.open()
		emit('popupClick')
	}
	const popupCancel = () => {
		popup.value.close()
	}
	const popupSubmit = () => {
		console.log(multiIndex)
		const data = {
			year: years.value[multiIndex.value[0]].year,
			month: months.value[multiIndex.value[1]].month,
			day: days.value[multiIndex.value[2]].day,
		}
		emit('popupSubmit', multiIndex.value, data)
	}
	const getYears = () => {
		uni.request({
			url: 'https://www.aiyccc.com/rili_year',
			method: 'POST',
			success: (res) => {
				console.log(res.data);
				years.value = res.data.data
				getMonths(years.value[0].year)
			}
		});
	}
	const getMonths = (year) => {
		uni.request({
			url: 'https://www.aiyccc.com/rili_month',
			method: 'POST',
			data: {
				year: year || currentYear.value
			},
			success: (res) => {
				console.log(res.data);
				months.value = res.data.data.sort((a, b) => a.month - b.month)
				getDays(year, nowMonth.value || 1)
			}
		});
	}
	const getDays = (year, month) => {
		uni.request({
			url: 'https://www.aiyccc.com/rili_day',
			method: 'POST',
			data: {
				year: year || currentYear.value,
				month: month || currentMonth.value
			},
			success: (res) => {
				console.log(res.data);
				days.value = res.data.data.sort((a, b) => a.day - b.day)
			}
		});
	}
	const transform = (data) => {
		
	}
	const handleNong = async () => {
		isShowNong.value = !isShowNong.value
		const year = years.value[multiIndex.value[0]].year
		const month = months.value[multiIndex.value[1]].month
		const day = days.value[multiIndex.value[2]].day

		if (isShowNong.value) {
			const p = {
				type: 1,
				year: year,
				month: month,
				day: day
			}
			uni.request({
				url:'https://www.aiyccc.com/rili_transformation',
				data:{
					...p
				},
				success:(res) => {
				 const {
				 	nyear,
				 	nmonth,
				 	nday
				 } = res.data.data
				 getMonths(years.value.find(item => item.nyear === nyear).year)
				 getDays(years.value.find(item => item.nyear === nyear).year, months.value.find(item => item.nmonth === nmonth).month)
				 const lYearIndex = years.value.findIndex(item => item.nyear === nyear)
				 const lMonthIndex = months.value.findIndex(item => item.nmonth === nmonth)
				 const lDayIndex = days.value.findIndex(item => item.nday === nday)
				 multiIndex.value = [lYearIndex, lMonthIndex, lDayIndex]
				 console.log(multiIndex.value)
				}
			})
			// const a = await transform(p)
			// console.log(111, a)
			// // 公历转农历
			// console.log(calendar.solar2lunar(year, month, day))
			// const {
			// 	lYear,
			// 	lMonth,
			// 	lDay,
			// 	lunarDate
			// } = calendar.solar2lunar(year, month, day)
			// getMonths(lYear)
			// getDays(lYear, lMonth)
			// const lYearIndex = years.value.findIndex(item => +item.year === +lYear)
			// const lMonthIndex = months.value.findIndex(item => +item.month === +lMonth)
			// const lDayIndex = days.value.findIndex(item => +item.day === +lDay)
			// multiIndex.value = [lYearIndex, lMonthIndex, lDayIndex]
			// console.log(multiIndex.value)
		} else {
			// 农历转公历 
			// 滑动农历之后转为公历
			const gyear = years.value[multiIndex.value[0]].year
			const gmonth = months.value[multiIndex.value[1]].month
			const gday = days.value[multiIndex.value[2]].day
			const {
				cYear,
				cMonth,
				cDay
			} = calendar.lunar2solar(lunarDate[0], lunarDate[1], lunarDate[2])
			const cYearIndex = years.value.findIndex(item => +item.year === +cYear)
			const cMonthIndex = months.value.findIndex(item => +item.month === +cMonth)
			const cDayIndex = days.value.findIndex(item => +item.day === +cDay)
			multiIndex.value = [cYearIndex, cMonthIndex, cDayIndex]
			console.log(calendar.lunar2solar(lunarDate[0], lunarDate[1], lunarDate[2]), multiIndex.value)
		}

	}
	const pickerChange = (e) => {
		console.log(e)
		const val = e.detail.value
		const year = val[0]
		const month = val[1]
		const day = val[2]
		nowYear.value = years.value[val[0]].year
		nowMonth.value = months.value[month].month
		console.log(nowYear.value, nowMonth.value, multiIndex.value[0], year, val)
		if (multiIndex.value[0] !== year) {
			getMonths(nowYear.value)
		}
		if (multiIndex.value[1] !== month) {
			getDays(nowYear.value, nowMonth.value)
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
