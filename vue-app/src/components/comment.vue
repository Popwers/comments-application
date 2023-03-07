<script setup lang="ts">
	const props = defineProps({ comment: Object });
</script>

<template>
	<div class="card">
		<div class="flex">
			<h2>{{ props.comment?.firstName }} {{ props.comment?.lastName }}</h2>
			<p class="date">
				{{
					new Date(Date.parse(props.comment?.birthDate)).toLocaleDateString('fr-FR', {
						weekday: 'long',
						year: 'numeric',
						month: 'long',
						day: 'numeric',
					})
				}}
			</p>
		</div>
		<p class="mail">{{ props.comment?.email }}</p>
		<p>{{ props.comment?.text }}</p>
	</div>
</template>

<style scoped>
	h2 {
		font-size: 1.5rem;
		margin: 0;
		margin-bottom: 0.2rem;
	}

	.date {
		font-size: 0.8rem;
		margin: 0;
	}

	.mail {
		font-size: 0.7rem;
		margin: 0;
	}

	.flex {
		display: flex;
		justify-content: space-between;
	}
	.card {
		background-color: #fff;
		border-radius: 0.5rem;
		box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
		padding: 1rem;
		margin-bottom: 1rem;
	}
</style>
