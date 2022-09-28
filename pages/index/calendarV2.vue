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
<script>
	export default {
		name: 'calendar',
		data() {
			return {
				popup: null, // 选择框
				years: [], // 年数组，公历和农历共用
				months: [], // 月数组， 公历和农历共用
				multiIndex: [0, 0, 0], // 时间选择索引[1,4,11] =》 1992，5，12 对应为3列索引
				currentDate: null, // 选择时间
				currentYear: null, // 当前年
				currentMonth: null, // 当前月
				currentDay: null, // 当前日
				nowYear: null, // 日期选择器选择年
				nowMonth: 1, // 日期选择其选择月
				days: [], // 日数组，公农历共用
				isShowNong: false, // 是否展示农历 ，默认不展示，
			}
		},
		mounted() {
			// 获取当前时间 因为没有正式数据，模拟 1992 5 12
			this.currentYear = 1922 || new Date().getFullYear()
			this.currentMonth = 5 || new Date().getMonth()
			this.currentDay = 12 || new Date().getDay()
			// 展示选择时间
			this.currentDate = {
				year: this.currentYear,
				month: this.currentMonth,
				day: this.currentDay
			}

			// 获取年 月 日数组，参数为true 是为了获取连续获取
			this.getYears(true)
		},
		methods: {
			popupClick() {
				this.$refs.popup.open()
			},
			// 关闭日期选择
			popupCancel() {
				this.$refs.popup.close()
			},
			// 日期选择提交
			popupSubmit() {
				// 根据 选择器绑定索引数组 获取选择年月日
				const data = {
					year: this.years[this.multiIndex[0]].year,
					month: this.months[this.multiIndex[1]].month,
					day: this.days[this.multiIndex[2]].day,
				}
				this.currentDate = {
					...data
				}
				this.$refs.popup.close()
				// 将数据传给父组件
				this.$emit('popupSubmit', this.multiIndex, data)
			},
			// 获取年数组
			getYears(flag) {
				uni.request({
					url: 'https://www.aiyccc.com/rili_year',
					method: 'POST',
					success: (res) => {
						if (res.data.data.length > 0) {
							this.years = res.data.data
							// 根据参数判断是否继续向下获取月 
							if (flag) {
								this.getMonths(this.currentYear || this.years[0].year, true)
							}
						} else {
							this.years = []
						}
					}
				});
			},
			// 获取公历月数组
			getMonths(year, flag) {
				uni.request({
					url: 'https://www.aiyccc.com/rili_month',
					method: 'POST',
					data: {
						year: year
					},
					success: (res) => {
						if (res.data.data.length > 0) {
							this.months = res.data.data
							// 根据参数判断是否继续向下获取日 
							if (flag) {
								this.getDays(this.currentYear || this.years[0].year, this.currentMonth || this
									.months[0].month, flag)
							}

						} else {
							this.months = []
						}

					}
				});
			},
			// 获取农历月数组
			getnMonths(year) {
				uni.request({
					url: 'https://www.aiyccc.com/rili_nmonth',
					method: 'POST',
					data: {
						year: year
					},
					success: (res) => {
						if (res.data.data.length > 0) {
							this.months = res.data.data
						} else {
							this.months = []
						}

					}
				});
			},
			// 获取日数组
			getDays(year, month, flag) {
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
							this.days = res.data.data
							// 根据参数判断是否继续向下获取日 
							if (flag) {
								// 根据当前日期自动锚点。仅作为初始化数据时使用
								this.$nextTick(() => {
									const currentYearIndex = this.years.findIndex(
										item => item
										.year === this.currentYear)
									const currentMonthIndex = this.months.findIndex(
										item =>
										item.month === this.currentMonth)
									const currentDayIndex = this.days.findIndex(
										item => item
										.day === this.currentDay)
									this.multiIndex = [currentYearIndex,
										currentMonthIndex,
										currentDayIndex
									]
								})

							}
						} else {
							this.days = []
						}
					}
				})

			},
			// 获取农历天
			getnDays(year, month) {
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
							this.days = res.data.data
						} else {
							this.days = []
						}
					}
				});
			},
			// 农历方法
			handleNong() {
				// 公农历互转接口可以不调用，通过选择的日拿到相对应的数据
				this.isShowNong = !this.isShowNong
				// 展示农历
				if (this.isShowNong) {
					// 获取公历数据
					const year = this.years[this.multiIndex[0]].year
					const month = this.months[this.multiIndex[1]].month
					const day = this.days[this.multiIndex[2]].day
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
							this.getnMonths(this.years.find(item => item.nyear === nyear).nyear)
							// 获取农历日数
							uni.request({
								url: 'https://www.aiyccc.com/rili_nday',
								method: 'POST',
								data: {
									year: this.years.find(item => item.nyear === nyear).nyear,
									month: this.months.find(item => item.nmonth === nmonth).nmonth
								},
								success: (res) => {
									if (res.data.data.length > 0) {
										console.log(res.data);
										this.days = res.data.data
										// 农历数据索引
										const lYearIndex = this.years.findIndex(item => item
											.nyear === nyear)
										const lMonthIndex = this.months.findIndex(item => item
											.nmonth ===
											nmonth)
										const lDayIndex = this.days.findIndex(item => item
											.nday === nday)
										// 索引数据
										this.multiIndex = [lYearIndex, lMonthIndex, lDayIndex]
										console.log(this.multiIndex)
									} else {
										this.days = []
									}
								}
							});

						}
					})
				} else {
					// 农历数据
					const nyear = this.years[this.multiIndex[0]].nyear
					const nmonth = this.months[this.multiIndex[1]].nmonth
					const nday = this.days[this.multiIndex[2]].nday
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
							this.getMonths(this.years.find(item => item.year === year).year)
							// 获取公历日数据
							uni.request({
								url: 'https://www.aiyccc.com/rili_day',
								method: 'POST',
								data: {
									year: this.years.find(item => item.year === year).year,
									month: this.months.find(item => item.month === month).month
								},
								success: (res) => {
									if (res.data.data.length > 0) {
										console.log(res.data);
										this.days = res.data.data
										// 处理索引
										const lYearIndex = this.years.findIndex(item => item
											.year === year)
										const lMonthIndex = this.months.findIndex(item => item
											.month === month)
										const lDayIndex = this.days.findIndex(item => item
											.day ===
											day)
										this.multiIndex = [lYearIndex, lMonthIndex, lDayIndex]
										console.log(this.multiIndex)
									} else {
										this.days = []
									}
								}
							});



						}
					})
				}
			},
			// 日期选择方法
			pickerChange(e) {
				console.log('我执行了', e)
				// 选择的索引
				const val = e.detail.value
				const year = val[0]
				const month = val[1]
				const day = val[2]
				// 根据索引获取 年月的数据
				this.nowYear = this.years[year]
				this.nowMonth = this.months[month]
				// 判断年索引是否变化，变化之后获取新的月日数据
				if (this.multiIndex[0] !== year) {
					// 判断是否是农历
					if (this.isShowNong) {
						this.getnMonths(this.nowYear.nyear)
						this.getnDays(this.nowYear.nyear, this.nowMonth.nmonth)
					} else {
						console.log(1)
						this.getMonths(this.nowYear.year)
						this.getDays(this.nowYear.year, this.nowMonth.month)
					}

				}
				if (this.multiIndex[1] !== month) {
					if (this.isShowNong) {
						this.getnDays(this.nowYear.nyear, this.nowMonth.nmonth)
					} else {
						this.getDays(this.nowYear.year, this.nowMonth.month)
					}
				}
				this.multiIndex = val

			},
		}
	}
</script>

<style>
	.item {
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>
