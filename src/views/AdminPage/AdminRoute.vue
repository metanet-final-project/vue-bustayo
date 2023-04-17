<template>
	<Navbar light />
	<div class="row">
		<Sidebar />
		<div class="col-md-9">
			<div>
				<div
					class="mt-4 d-flex justify-content-between"
					style="background-color: #f0f2f5; border-radius: 10px"
				>
					<h4 class="m-3 p-2">회원관리</h4>
					<MaterialButton class="m-3" variant="contained" color="dark"
						>회원등록</MaterialButton
					>
				</div>
				<div class="container mt-4">
					<div class="row">
						<div class="col-lg-12">
							<table class="table">
								<thead>
									<tr class="text-center">
										<th class="col-1">번호</th>
										<th class="col-2">아이디</th>
										<th class="col-2">이름</th>
										<th class="col-3">이메일</th>
										<th class="col-3">전화번호</th>
										<th class="col-1">관리</th>
									</tr>
								</thead>
								<tbody class="table-group-divider text-center">
									<tr v-for="(member, idx) in memberList" :key="member.id">
										<td>{{ idx + 1 }}</td>
										<td>{{ member.loginId }}</td>
										<td>{{ member.name }}</td>
										<td>{{ member.email }}</td>
										<td>{{ member.phone }}</td>
										<td>
											<MaterialButton
												color="light"
												class="input-group-static btn-sm mb-0"
												@click="goManageMember(member.id)"
												>관리</MaterialButton
											>
										</td>
									</tr>
								</tbody>
							</table>
						</div>
						<div
							v-if="showModal"
							class="modal"
							tabindex="-1"
							style="display: flex"
						>
							<div class="modal-dialog">
								<div
									class="modal-content"
									style="
										box-shadow: 0 14px 28px rgba(0, 0, 0, 0.25),
											0 10px 10px rgba(0, 0, 0, 0.22);
										width: 400px;
									"
								>
									<div class="modal-header">
										<h5 class="modal-title">회원관리</h5>
										<MaterialBadge
											color="dark"
											rounded
											class="text-white"
											@click.prevent="showModal = false"
											style="cursor: pointer"
										>
											닫기
										</MaterialBadge>
									</div>
									<div class="modal-body">
										<form role="form" class="text-start p-3">
											<label>회원고유번호</label>
											<div class="input-group input-group-outline mb-2">
												<input
													type="text"
													class="form-control"
													disabled
													v-model="member.id"
												/>
											</div>
											<label>아이디</label>
											<div class="input-group input-group-outline mb-2">
												<input
													type="text"
													class="form-control"
													disabled
													v-model="member.loginId"
												/>
											</div>
											<label>이름</label>
											<div class="input-group input-group-outline mb-2">
												<input
													type="text"
													class="form-control"
													v-model="member.name"
												/>
											</div>
											<label>이메일</label>
											<div class="input-group input-group-outline mb-2">
												<input
													type="text"
													class="form-control"
													v-model="member.email"
												/>
											</div>
											<label>전화번호</label>
											<div class="input-group input-group-outline mb-2">
												<input
													type="text"
													class="form-control"
													v-model="member.phone"
												/>
											</div>
											<!-- <label>권한</label>
											<div class="input-group input-group-outline">
												<input
													type="text"
													class="form-control"
													v-model="member.role"
													disabled
												/>
											</div> -->
											<div class="modal-footer justify-content-between">
												<MaterialButton
													variant="contained"
													color="dark"
													class="mb-0"
													@click.prevent="updateMember"
													>수정하기</MaterialButton
												>
												<MaterialButton
													variant="contained"
													color="warning"
													class="mb-0"
													@click.prevent="deleteMember(member.id)"
													>회원삭제</MaterialButton
												>
											</div>
										</form>
									</div>
								</div>
							</div>
						</div>
						<div class="row">
							<div class="col-3 offset-md-9">
								<div class="input-group input-group-outline my-3">
									<label class="form-label">회원검색</label>
									<input
										type="text"
										class="form-control form-control-md"
										placeholder
										isrequired="false"
									/>
								</div>
							</div>
						</div>
						<section class="py-4">
							<div class="container">
								<div class="row justify-space-between py-2">
									<div class="col-lg-4 mx-auto">
										<MaterialPagination :color="light">
											<MaterialPaginationItem prev class="ms-auto" />
											<MaterialPaginationItem label="1" active />
											<MaterialPaginationItem label="2" />
											<MaterialPaginationItem label="3" />
											<MaterialPaginationItem label="4" />
											<MaterialPaginationItem label="5" />
											<MaterialPaginationItem next />
										</MaterialPagination>
									</div>
								</div>
							</div>
						</section>
					</div>
				</div>
			</div>
		</div>
	</div>
	<Footer />
</template>

<script setup>
import Footer from '@/layouts/Footer.vue';
import Navbar from '@/layouts/Navbar.vue';
import Sidebar from './Sections/SideBar.vue';
import MaterialButton from '@/components/MaterialButton.vue';
import axios from 'axios';
import { ref } from 'vue';
import MaterialPagination from '@/components/MaterialPagination.vue';
import MaterialPaginationItem from '@/components/MaterialPaginationItem.vue';
import setMaterialInput from '@/assets/js/material-input';
import { onMounted } from 'vue';
import MaterialBadge from '@/components/MaterialBadge.vue';
import Swal from 'sweetalert2';

onMounted(() => {
	setMaterialInput();
});

const member = ref();
const memberList = ref([]);
const showModal = ref(false);

const findAllMember = async () => {
	try {
		const result = await axios.get('/api/member/findAllMember');
		if (result.data != null) {
			memberList.value = result.data;
		}
	} catch (error) {
		console.error(error);
	}
};
findAllMember();

const goManageMember = async memberId => {
	try {
		const result = await axios.get(`/api/member/findById/${memberId}`);
		if (result.data != null) {
			member.value = result.data;
		}
	} catch (error) {
		console.error(error);
	}
	showModal.value = true;
};

const updateMember = async () => {
	try {
		await axios.put(`/api/member/update`, member.value);
		showToast('success', '회원수정을 완료했습니다.');
		showModal.value = false;
	} catch (error) {
		console.error(error);
		showToast('error', '회원수정에 실패했습니다.');
	}
};

const deleteMember = async memberId => {
	try {
		await axios.delete(`/api/member/delete/${memberId}`);
		showToast('success', '회원삭제를 완료했습니다.');
		showModal.value = false;
	} catch (error) {
		console.error(error);
		showToast('error', '회원삭제에 실패했습니다.');
	}
};

const Toast = Swal.mixin({
	toast: true,
	position: 'bottom-end',
	showConfirmButton: false,
	timer: 3000,
	timerProgressBar: true,
});

const showToast = (icon, title) => {
	Toast.fire({
		icon: icon,
		title: title,
	});
};
</script>