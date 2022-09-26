<template>
	<view>
		<view @click="popupClick">1231</view>
		<uni-popup ref="popup" type="center">
			<view style="width: 700rpx;height: 500rpx;background-color: azure;">
				<picker-view indicator-style="height: 50px;" style="width: 100%; height: 300px;" :value="multiIndex" @change="pickerChange"
								class="picker-view">
								<picker-view-column>
									<view class="item" v-for="(item,index) in years" :key="index">{{item.year}}({{item.nyear}})</view>
								</picker-view-column>
								<picker-view-column>
									<view class="item" v-for="(item,index) in months" :key="index">{{item.month}}({{item.nmonth}})</view>
								</picker-view-column>
								<picker-view-column>
									<view class="item" v-for="(item,index) in days" :key="index">{{item.day}}({{item.nday}})</view>
								</picker-view-column>
							</picker-view>
			</view>
			
		</uni-popup>
	</view>
</template>

<script setup>
import { onMounted, ref } from "vue";
	const popup = ref(null)
	const years = ref([])
	const months = ref([])
	const multiIndex = ref([0,0,0])
	const currentYear = ref(null)
	const currentMonth = ref(null)
	const days = ref([])
	onMounted(() => {
		currentYear.value = new Date().getFullYear()
		currentMonth.value = new Date().getMonth()
		console.log('ref',popup)
		getYears()
	})
	const popupClick = () => {
		popup.value.open()
	}
	const getYears = () => {
		uni.request({
		    url: 'https://www.aiyccc.com/rili_year',
				method: 'POST',
		    success: (res) => {
		        console.log(res.data);
						years.value = res.data.data
						getMonths()
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
							months.value = res.data.data
							getDays(2021, 1)
			    }
			});
	}
	const getDays = (year, month) => {
		uni.request({
			url: 'https://www.aiyccc.com/rili_day',
			method: 'POST',
			data: {
				year: year || currentYear.value,
				month: month || currentMonth.value + 1
			},
			success: (res) => {
			        console.log(res.data);
							days.value = res.data.data
			    }
			});
	}
</script>

<style>
</style>