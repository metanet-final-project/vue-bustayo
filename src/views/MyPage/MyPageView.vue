<template>
	<Navbar transparent />
	<Header>
		<div
			class="page-header min-height-400"
			:style="{ backgroundImage: `url(${image})` }"
			loading="lazy"
		>
			<span class="mask bg-gradient-dark opacity-6"></span>
			<div class="overlay-text">
				<h1>마이페이지</h1>
			</div>
		</div>
	</Header>
	<!-- <div class="card card-body blur shadow-blur mx-3 mx-md-4 mt-n6 mb-4">
		<Profile /> 
	</div> -->
	<div class="card-wrapper">
		<div class="card-deck d-flex justify-content-center">
			<div class="card text-center mx-3" style="width: 23rem">
				<div class="card-body">
					<h5 class="card-title">나의 예매내역</h5>
					<p class="card-text">{{ myBookingList.length }}건</p>
					<a href="/mypage/booking" class="btn btn-dark">조회</a>
				</div>
			</div>
			<div class="card text-center mx-3" style="width: 23rem">
				<div class="card-body">
					<h5 class="card-title">내 정보 수정</h5>
					<p class="card-text">회원정보 수정</p>
					<a href="/mypage/modify" class="btn btn-dark">수정</a>
				</div>
			</div>
			<div class="card text-center mx-3" style="width: 23rem">
				<div class="card-body">
					<h5 class="card-title">결제 내역</h5>
					<p class="card-text">결제내역 조회</p>
					<a href="/mypage/pay" class="btn btn-dark">조회</a>
				</div>
			</div>
		</div>
	</div>
	<Footer />
	<MemberAuth />
</template>

<script setup>
import Navbar from '@/layouts/Navbar.vue';
import Header from '@/examples/Header.vue';
import Footer from '@/layouts/Footer.vue';
import image from '@/assets/img/busimage.png';
import axios from 'axios';
import { ref } from 'vue';
import MemberAuth from '@/layouts/Auth/MemberAuth.vue';

let myBookingList = ref('');

const getMyBookingList = async () => {
	const res = await axios.get(
		`/api/booking/find/findValidByLoginId/${localStorage.getItem('loginId')}`,
	);
	myBookingList.value = res.data;
	console.log(
		'myBookingList' +
			myBookingList.value +
			' ' +
			localStorage.getItem('loginId'),
	);
};
getMyBookingList();
</script>

<style scoped>
.overlay-text {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	z-index: 1;
	color: #fff;
	text-align: center;
}

.overlay-text h1 {
	color: #fff;
}

.card-wrapper {
	margin-top: 100px;
}
</style>
