<template>
	<section class="my-5 pt-5">
		<div class="container">
			<div class="row">
				<div class="col-md-6 m-auto">
					<h3>승차권 예매</h3>
					<p class="mb-4 fw-bolder">
						예매 가능한 좌석정보와 운행정보, 간펀한 결제 방식으로 온라인 승차권
						예매를 하실 수 있습니다. <br />시외버스 예매 시스템으로 안전하고
						편리하게 여행하세요.
					</p>
					<div class="row">
						<ModalStarting @stId="startingTerminal" />
						<ModalEnding @et="endingTerminal" :variable="parentVariable" />
					</div>
					<div class="row mt-3">
						<div class="col-6">
							<div class="input-group input-group-static my-3">
								<label>가는일</label>
								<input
									type="date"
									class="form-control"
									v-model="selectedDate"
									@change="compareDates"
								/>
							</div>
						</div>
					</div>
					<div class="row mt-3">
						<div class="text-center">
							<MaterialButton
								class="my-4 mb-2"
								variant="contained"
								color="dark"
								fullWidth
								@click="goSchedule"
								>배차조회</MaterialButton
							>
						</div>
					</div>
				</div>
				<div class="col-md-4 m-auto">
					<h3>비회원 예매조회</h3>
					<p class="mb-4 fw-bolder">
						비회원으로 예매하셨나요?<br />
						비회원 예매조회로 예매 내역을 빠르게 조회해보세요.
					</p>
					<div class="row mt-3 col-11 m-auto">
						<div>
							<div class="input-group input-group-static">
								<label>전화번호</label>
								<input
									type="tel"
									class="form-control"
									id="nonMemPhone"
									v-model="nonMember.phone"
									@input="formatPhone"
									maxlength="13"
									:placeholder="isFocused ? 'XXX-XXXX-XXXX' : ''"
									@focus="isFocused = true"
									@blur="isFocused = false"
								/>
							</div>
						</div>
					</div>
					<div class="row mt-3 col-11 m-auto">
						<div>
							<div class="input-group input-group-static my-3">
								<label>생년월일</label>
								<input
									type="date"
									class="form-control"
									v-model="nonMember.birthStr"
								/>
							</div>
						</div>
					</div>
					<div class="row mt-3">
						<div class="text-center">
							<MaterialButton
								class="my-4 mb-2"
								variant="contained"
								color="dark"
								fullWidth
								@click="goNonMemberBooking"
								>예매조회</MaterialButton
							>
						</div>
					</div>
				</div>
			</div>
		</div>
	</section>
</template>

<script setup>
import { onMounted } from 'vue';
import { ref } from 'vue';
import MaterialButton from '@/components/MaterialButton.vue';
import setMaterialInput from '@/assets/js/material-input';
import ModalStarting from '@/components/ModalStarting.vue';
import ModalEnding from '@/components/ModalEnding.vue';
import axios from 'axios';
import { useRouter } from 'vue-router';
import Swal from 'sweetalert2';

onMounted(() => {
	setMaterialInput();
});

const router = useRouter();
const stId = ref(); // 출발지 터미널 정보
const endId = ref(); // 도착지 터미널 정보
let parentVariable = stId; // 출발지 정보 props 전달
const searchInfo = ref({}); // 배차조회에 사용될 검색 조건
const routeId = ref();
const nonMember = ref({});

const formatPhone = event => {
	let input = event.target.value.replace(/-/g, '').replace(/\D/g, '');
	let formattedInput = '';
	if (input.length > 3) {
		formattedInput += input.slice(0, 3) + '-';
		if (input.length > 7) {
			formattedInput += input.slice(3, 7) + '-';
			formattedInput += input.slice(7, 11);
		} else {
			formattedInput += input.slice(3, 7);
		}
	} else {
		formattedInput += input;
	}
	event.target.value = formattedInput;
};

const startingTerminal = async id => {
	const result = await axios.get(`/api/terminal/findById/${id}`);
	stId.value = result.data;
};
const selectedDate = ref();
const endingTerminal = async id => {
	const result = await axios.get(`/api/terminal/findById/${id}`);
	endId.value = result.data;
};

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
		searchInfo.value.stDate = selectedDate.value;
	}
};

const goSchedule = async () => {
	if (
		stId.value == undefined ||
		endId.value == undefined ||
		searchInfo.value.stDate == undefined
	) {
		showToast('warning', '검색조건을 선택해주세요.');
		return;
	}
	const result = await axios.get(
		`/api/route/find/start-end/point/${stId.value.id}/${endId.value.id}`,
	);
	routeId.value = result.data;

	router.push({
		name: 'ScheduleSearch',
		query: {
			routeId: routeId.value,
			date: searchInfo.value.stDate,
		},
	});
};

const goNonMemberBooking = async () => {
	try {
		const result = await axios.post(
			'/api/non-member/find/info',
			nonMember.value,
		);
		if (result.data != null) {
			console.log(result.data);
			router.push({
				name: 'NonMemBooking',
				query: {
					nonMemberId: result.data,
				},
			});
		}
	} catch (error) {
		console.error(error);
		showToast('warning', '올바른 정보를 입력해주세요.');
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
