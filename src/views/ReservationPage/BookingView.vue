<template>
	<Navbar transparent />
	<Header>
		<div
			class="page-header min-height-400"
			:style="{ backgroundImage: `url(${image})` }"
			loading="lazy"
		>
			<div class="container">
				<div class="row justify-content-center">
					<div class="col-lg-8 text-center mx-auto my-auto">
						<h1 class="text-white">
							<span class="text-white" id="typed"></span>
						</h1>
						<p class="lead mb-4 text-white">시외버스예매</p>
					</div>
				</div>
			</div>
			<span class="mask bg-gradient-dark opacity-8"></span>
		</div>
	</Header>
	<div class="card card-body blur shadow-blur mx-3 mx-md- mt-n6 mb-4">
		<Agree />
		<Ticket />
	</div>
	<Footer />
</template>

<script setup>
import Navbar from '@/layouts/Navbar.vue';
import Header from '@/examples/Header.vue';
import Footer from '@/layouts/Footer.vue';
import Ticket from './Sections/Ticket.vue';
import Agree from './Sections/Agree.vue';
import image from '@/assets/img/busimage.png';
</script>

<style scoped>
p {
	font-size: 50px;
	font-weight: bold;
}
</style>
