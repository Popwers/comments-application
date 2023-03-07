<script setup lang="ts">
	import { ref, onMounted } from 'vue';
	import VueDatePicker from '@vuepic/vue-datepicker';
	import '@vuepic/vue-datepicker/dist/main.css';
	import Comment from './components/comment.vue';

	const darkMode = ref(false);
	darkMode.value = window.matchMedia('(prefers-color-scheme: dark)').matches;
	window
		.matchMedia('(prefers-color-scheme: dark)')
		.addEventListener('change', e => (darkMode.value = e.matches));

	const comments = ref([] as any[]);
	const formData = ref({
		firstName: {
			value: '',
			error: false,
			message: '',
		},
		lastName: {
			value: '',
			error: false,
			message: '',
		},
		birthDate: {
			value: new Date().toDateString(),
			error: false,
			message: '',
		},
		email: {
			value: '',
			error: false,
			message: '',
		},
		comment: {
			value: '',
			error: false,
			message: '',
		},
	});

	const fetchData = async () => {
		const request = await fetch('https://localhost:4443/api/comments');
		const respond = await request.json();
		comments.value = respond['hydra:member'].reverse();
	};

	const sendData = async () => {
		const { email, firstName, lastName, birthDate, comment } = formData.value;

		Object.values(formData.value).forEach(field => {
			field.error = false;
			field.message = '';
		});

		try {
			const request = await fetch('https://localhost:4443/api/comments', {
				method: 'POST',
				body: JSON.stringify({
					email: email.value,
					firstName: firstName.value,
					lastName: lastName.value,
					birthDate: birthDate.value,
					text: comment.value,
				}),
				headers: {
					'Content-type': 'application/json; charset=UTF-8',
				},
			});

			const respond = await request.json();

			if (
				respond['hydra:description'] ===
				'The data is either an empty string or null, you should pass a string that can be parsed with the passed format or a valid DateTime string.'
			) {
				formData.value.birthDate.error = true;
				formData.value.birthDate.message = 'Veuillez entrer une date valide';
			}

			if (respond.violations) {
				respond.violations.forEach((violation: { propertyPath: string; message: string }) => {
					switch (violation.propertyPath) {
						case 'email':
							formData.value.email.error = true;
							formData.value.email.message = violation.message;
							break;
						case 'firstName':
							formData.value.firstName.error = true;
							formData.value.firstName.message = violation.message;
							break;
						case 'lastName':
							formData.value.lastName.error = true;
							formData.value.lastName.message = violation.message;
							break;
						case 'birthDate':
							formData.value.birthDate.error = true;
							formData.value.birthDate.message = violation.message;
							break;
						case 'text':
							formData.value.comment.error = true;
							formData.value.comment.message = violation.message;
							break;
						default:
							console.log('Unknown error');
					}
				});
			}

			if (respond.id) {
				formData.value.comment.value = '';
				fetchData();
			}
		} catch (error) {
			formData.value.comment.error = true;
			formData.value.comment.message = 'Une erreur est survenue ! ';
		}
	};

	onMounted(fetchData);
</script>

<template>
	<h1>Comments App</h1>

	<form @submit.prevent="sendData">
		<div class="form-group">
			<div>
				<label for="firstName">Prénom</label>
				<input
					id="firstName"
					@click="formData.firstName.error = false"
					:class="{ error: formData.firstName.error }"
					v-model="formData.firstName.value"
				/>
				<p class="error-message" v-show="formData.firstName.error">
					{{ formData.firstName.message }}
				</p>
			</div>
			<div>
				<label for="lastName">Nom</label>
				<input
					id="lastName"
					@click="formData.lastName.error = false"
					:class="{ error: formData.lastName.error }"
					v-model="formData.lastName.value"
				/>
				<p class="error-message" v-show="formData.lastName.error">
					{{ formData.lastName.message }}
				</p>
			</div>
		</div>
		<div class="form-group">
			<div>
				<label for="birthDate">Date de naissance</label>
				<VueDatePicker
					id="birthDate"
					:class="{ error: formData.birthDate.error }"
					v-model="formData.birthDate.value"
					:enable-time-picker="false"
					:dark="darkMode"
					auto-apply
				></VueDatePicker>
				<p class="error-message" v-show="formData.birthDate.error">
					{{ formData.birthDate.message }}
				</p>
			</div>
			<div>
				<label for="email">E-mail</label>
				<input
					id="email"
					@click="formData.email.error = false"
					:class="{ error: formData.email.error }"
					v-model="formData.email.value"
				/>
				<p class="error-message" v-show="formData.email.error">
					{{ formData.email.message }}
				</p>
			</div>
		</div>

		<label for="comment">Commentaire</label>
		<textarea
			id="comment"
			@click="formData.comment.error = false"
			:class="{ error: formData.comment.error }"
			v-model="formData.comment.value"
		/>
		<p class="error-message" v-show="formData.comment.error">
			{{ formData.comment.message }}
		</p>

		<button>Envoyer</button>
	</form>

	<div class="comment-wrapper">
		<Comment v-for="comment in comments" :key="comment.id" :comment="comment" />
	</div>
</template>

<style scoped>
	h1 {
		text-align: center;
	}

	form {
		width: 60vw;
		margin: 0 auto;
		text-align: center;
	}

	label {
		display: block;
		margin: 0.5rem;
	}

	textarea {
		display: block;
		box-sizing: border-box;
		width: 60vw;
		min-height: 200px;
		margin: 0.5rem auto;
	}

	.form-group {
		display: flex;
		flex-direction: row;
		justify-content: center;
	}

	.form-group div {
		margin: 0.5rem;
	}

	.comment-wrapper {
		margin-top: 2rem;
	}
</style>
