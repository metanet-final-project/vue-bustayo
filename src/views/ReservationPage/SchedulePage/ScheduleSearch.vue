<template>
	<Navbar transparent />
	<Header>
		<div
			class="page-header min-height-400"
			:style="{ backgroundImage: `url(${image})` }"
			loading="lazy"
		>
			<span class="mask bg-gradient-dark opacity-8"></span>
		</div>
	</Header>
	<div class="page" id="seatChcPage">
		<h3>배차 조회</h3>

		<div class="selectSeat_wrap">
			<!-- compareBox -->
			<div class="compare_wrap">
				<!-- 좌측 infoBox -->
				<div class="infoBox">
					<p class="date" id="satsDeprDtm">
						{{
							scheduleInfo[0].startTime != null
								? moment(scheduleInfo[0].startTime)
										.locale('ko')
										.format('YYYY년 MM월 DD일')
								: ''
						}}
					</p>
					<div class="route_wrap" id="satsRotInfo">
						<div class="inner">
							<p class="roundBox departure" id="satsDeprTmlNm">
								{{ scheduleInfo[0].routeDTO.startTerminal.name }}
							</p>
							<!-- 출발지 -->

							<p class="roundBox arrive" id="satsArvlTmlNm">
								{{ scheduleInfo[0].routeDTO.endTerminal.name }}
							</p>
							<!-- 도착지 -->
						</div>
						<div class="detail_info">
							<span id="satsTakeDrtm"
								>{{ scheduleInfo[0].routeDTO.travelTime }}분 예정</span
							>
						</div>
					</div>
					<div class="date_select">
						<input
							type="date"
							class="form-control"
							v-model="selectedDate"
							@change="compareDates"
						/>
					</div>
				</div>
			</div>
			<!-- //좌측 infoBox -->

			<!-- 우측 detailBox -->
			<div class="main_box">
				<div class="date_box">
					<span>{{
						scheduleInfo[0].startTime != null
							? moment(scheduleInfo[0].startTime)
									.locale('ko')
									.format('YYYY년 MM월 DD일')
							: ''
					}}</span>
				</div>
				<div class="bustime_box">
					<div class="bustime_title">
						<table class="table_box">
							<tr class="table_title">
								<th>출발</th>
								<th>고속사</th>
								<th>등급</th>
								<th>가격</th>
								<th>잔여석</th>
								<th></th>
							</tr>
						</table>
					</div>
					<div class="bustime_wrap">
						<template v-for="schedule in scheduleInfo" :key="schedule.id">
							<table
								class="table_box"
								:class="schedule.countSeat == 22 ? 'sold_out' : ''"
							>
								<tr class="table_body" @click="pageMove(schedule)">
									<td>
										{{
											schedule.startTime != null
												? moment(schedule.startTime)
														.locale('ko')
														.format('HH:MM')
												: ''
										}}
									</td>
									<td>{{ schedule.busDTO.companyDTO.name }}</td>
									<td>{{ schedule.busDTO.grade }}</td>
									<td>{{ schedule.price }} 원</td>
									<td>{{ 22 - schedule.countSeat }} 석</td>
									<td v-if="schedule.countSeat != 22" class="btn_arrow">
										선택
									</td>
									<td v-if="schedule.countSeat == 22">매진</td>
								</tr>
							</table>
						</template>
					</div>
				</div>
			</div>
		</div>
	</div>
	<Footer />
</template>

<script setup>
import Navbar from '@/layouts/Navbar.vue';
import Header from '@/examples/Header.vue';
import Footer from '@/layouts/Footer.vue';
import image from '@/assets/img/busimage.png';
import { onMounted } from 'vue';
import 'v-calendar/style.css';

import { ref, watch } from 'vue';
import axios from 'axios';
import { useRoute, useRouter } from 'vue-router';
import Swal from 'sweetalert2';
import moment from 'moment';
import 'moment/locale/ko';
moment.locale('ko');
const route = useRoute();
const router = useRouter();
const date = ref(new Date());
const schedule = ref({
	routeId: route.query.routeId,
	date: route.query.date,
});
const scheduleInfo = ref([
	{
		id: null,
		routeDTO: {
			id: null,
			startTerminal: {
				id: null,
				name: null,
				location: null,
			},
			endTerminal: {
				id: null,
				name: null,
				location: null,
			},
			travelTime: null,
		},
		startTime: null,
		endTime: null,
		busDTO: {
			id: null,
			busNum: null,
			companyDTO: {
				id: null,
				name: null,
				phone: null,
			},
			grade: null,
		},
		price: null,
		countSeat: null,
		date: null,
	},
]);

//흐름
onMounted(() => {
	stream();
});

const stream = () => {
	if (route.query.routeId == null || route.query.date == null) {
		showToast('warning', '올바른 접근이 아닙니다.');
		router.push({ name: 'Home' });
	}
};

const selectedDate = ref();

const compareDates = () => {
	const inputDate = new Date(selectedDate.value); // 입력된 날짜를 Date 객체로 변환
	const currentDate = new Date(); // 현재 날짜와 시간을 가져옴

	if (inputDate < currentDate) {
		Swal.fire({
			title: '이전 날짜는 선택할 수 없습니다!',
			icon: 'error',
		});
		selectedDate.value = null;
	} else {
		date.value = selectedDate.value;
	}
};

const selectDate = () => {
	schedule.value.date = date.value;
	router.push({
		query: { routeId: route.query.routeId, date: date.value },
	});
	getSchedule();
};

const getSchedule = async () => {
	console.log(schedule.value);

	const result = await axios.get(
		`/api/schedule/find/seat/${schedule.value.routeId}/${schedule.value.date}`,
	);
	if (result.data == null || result.data.length == 0) {
		Swal.fire({
			title: '조회된 배차가 없습니다.',
			icon: 'error',
		});
		router.push({
			name: 'Home',
		});
	}
	scheduleInfo.value = result.data;
	console.log(scheduleInfo.value);
}; // 배차 정보

watch(date, selectDate);

getSchedule();

const pageMove = schedule => {
	console.log(schedule);
	if (schedule.countSeat == 22) {
		return;
	} else {
		router.push({
			name: 'ScheduleSeat',
			query: { id: schedule.id, routeId: schedule.routeDTO.id },
		});
	}
};

const Toast = Swal.mixin({
	toast: true,
	position: 'bottom-end',
	showConfirmButton: false,
	timer: 2000,
});

const showToast = (icon, title) => {
	Toast.fire({
		icon: icon,
		title: title,
	});
};
</script>

<style scoped>
.page {
	width: 100%; /* 부모 요소의 100% 너비 */
	max-width: 1000px; /* 최대 너비 제한 */
	height: 700px;
	margin: 0 auto; /* 가운데 정렬 */
	padding: 40px; /* 여백 추가 */
}

.selectSeat_wrap {
	width: 100%; /* 부모 요소의 100% 너비 */
	height: 100%; /* 높이 설정 */
	display: flex;
	flex-direction: row;
}

.page h3 {
	text-align: center;
}

.compare_wrap {
	padding: 10px;
	width: 27%;
	height: 100%;
	display: flex;
	flex-direction: row;
	background-color: #68b3ce;
	color: white;
	margin-right: 10px;
}

.compare_wrap p {
	font-weight: 700;
}

.route_wrap .departure:before {
	content: '출발지';
}

.route_wrap .arrive:before {
	content: '도착지';
}

.sold_out tr td {
	color: #ccc;
}

.route_wrap .roundBox:before {
	display: block;
	position: absolute;
	left: 20px;
	top: 0;
	width: 35px;
	height: 35px;
	border-radius: 50%;
	background-color: #fff;
	color: #74b3c7;
	font-size: 12px;
	text-align: center;
	line-height: 35px;
}

.route_wrap .roundBox {
	display: block;
	overflow: hidden;
	position: relative;
	min-height: 35px;
	padding-left: 67px;
	font-size: 20px;
	color: #f7f7f7;
	white-space: nowrap;
	text-overflow: ellipsis;
}

.form-control {
	background-color: white;
	width: 150px;
	margin: auto;
	padding: 5px 10px;
}

.form-control:hover {
	background-color: white;
}

.infoBox {
	width: 100%;
}

.date_select {
	width: 100%;
	height: 100%;
	margin: 10px auto;
	text-align: center;
}

.box {
	width: 100%;
	height: 93%;
	display: flex;
	flex-direction: row;
	border: 1px solid black;
}

.seat {
	font-size: 20px;
	font-weight: 600;
	padding: 5px;
	text-align: center;
	border: 1px solid black;
}

.count_seat {
	text-align: center;
}

.detail_info {
	margin-bottom: 10px;
}

.detail_info span {
	text-align: end;
	font-size: 12px;
	display: block;
	padding-right: 30px;
}

.detail_info span:nth-child(0) {
	margin-top: 4px;
}

.route_wrap {
	border-bottom: 1px solid #ffffff;
	margin-top: 30px;
	font-size: 15px;
}

.route_wrap th {
	font-weight: 400;
	padding-left: 20px;
	font-size: 13px;
}
.main_box {
	width: 73%;
	height: 100%;
}
.date_box {
	font-size: 20px;
	font-weight: 600;
	padding: 5px;
	text-align: center;
	border: 1px solid black;
}
.schedule_box {
	width: 100%;
	height: 93%;
	display: flex;
	flex-direction: row;
	border: 1px solid black;
}

.bustime_box {
	width: 100%;
	height: 93%;
	border: 1px solid black;
	padding: 0px 5px;
	margin: auto;
}

.bustime_title {
	width: 97%;
	margin: auto;
}

.bustime_wrap {
	max-height: 93%; /* 최대 높이 설정 */
	overflow-y: auto; /* 세로 스크롤바 추가 */
	height: 93%;
	width: 97%;
	margin: auto;
}

.table_box {
	width: 95%;
}

.table_head,
.table_title {
	height: 40px;
	border-bottom: 1px solid #e6e6e6;
}
.table_title th {
	width: 90px;
}
.table_body td {
	width: 90px;
}

.table_body {
	height: 40px;
	border-bottom: 1px solid #e6e6e6;
}

.table_body:hover {
	cursor: pointer;
}

.table_body td:nth-child(6) {
	text-align: center; /* 가운데 정렬 */
	color: #68b3ce;
}
.table_body td {
	color: #333;
}

.btn_arrow {
	color: #68b3ce;
	background: url(https://www.kobus.co.kr/images/common/btn_arrow.png) right
		no-repeat;
}
</style>
