<template>
	<form @submit.prevent="onsubmit()">
		<va-input
			v-model="email"
			class="mb-3"
			type="email"
			:label="t('auth.email')"
			:error="!!emailErrors.length"
			:error-messages="emailErrors"
		/>

		<va-input
			v-model="password"
			class="mb-3"
			type="password"
			:label="t('auth.password')"
			:error="!!passwordErrors.length"
			:error-messages="passwordErrors"
		/>

		<div class="auth-layout__options d-flex align-center justify-space-between">
			<va-checkbox
				v-model="agreedToTerms"
				class="mb-0"
				:error="!!agreedToTermsErrors.length"
				:error-messages="agreedToTermsErrors"
			>
				<template #label>
					<span class="ml-2">
						{{ t('auth.agree') }}
						<span class="va-link">{{ t('auth.termsOfUse') }}</span>
					</span>
				</template>
			</va-checkbox>
			<router-link class="ml-1 va-link" :to="{ name: 'recover-password' }">
				{{ t('auth.recover_password') }}
			</router-link>
		</div>

		<div class="d-flex justify-center mt-3">
			<va-button class="my-0" @click="onsubmit">{{ t('auth.sign_up') }}</va-button>
		</div>
	</form>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import { useI18n } from 'vue-i18n'
import { useToast } from 'vuestic-ui'
import axios, { AxiosError } from 'axios'

const { init: toast } = useToast()

const { t } = useI18n()
const router = useRouter()
const email = ref('')
const password = ref('')
const agreedToTerms = ref(false)
const emailErrors = ref<string[]>([])
const passwordErrors = ref<string[]>([])
const agreedToTermsErrors = ref<string[]>([])

const REGISTER_SUCCESS = 1
const REGISTER_FAILED = 0

const formReady = computed(() => {
	return !(emailErrors.value.length || passwordErrors.value.length || agreedToTermsErrors.value.length)
})
// TODO: generate unique API_key for each user
const validate = async (username: string, password: string) => {
	const serverUrl = import.meta.env.VITE_CVERIFY_SERVER_URL === '' ? 'http://localhost:6969' : import.meta.env.VITE_CVERIFY_SERVER_URL;

	const apiKey = '1234' // TODO: Change this
	const endPointUrl = serverUrl + `/auth/register?api_key=${apiKey}`;
	const data = { username: username, password: password }
	axios
		.post(endPointUrl, data)
		.then((response) => {
			if (response.status === 200) {
				if (response.data['code'] === REGISTER_SUCCESS) {
					toast({
						message: 'Successfully registered!',
						color: 'success',
					})
					router.push({ name: 'dashboard' })
				} else if (response.data['code'] === REGISTER_FAILED) {
					toast({
						message: `Failed to register: ${response.data['message']}`,
						color: 'danger',
					})
					} else {
					toast({
						message: 'Unkown Error',
						color: 'danger',
					})
				}
			}
		})
		.catch((err: AxiosError | Error) => {
			if (axios.isAxiosError(err)) {
				if (err.status === 400) {
					toast({ message: `Failed to register: BAD REQUEST`, color: 'danger' })
				} else if (err.status == 422) {
					toast({ message: `Failed to register: INVALID EMAIL/PASSWORD`, color: 'danger' })
				} else {
					toast({ message: `Failed to register. is the server up?`, color: 'danger' })
					}
			} else {
				toast({ message: `Unkown Error`, color: 'danger' })
			}
		})
}

function onsubmit() {
	if (!formReady.value) return

	emailErrors.value = email.value ? [] : ['Email is required']
	passwordErrors.value = password.value ? [] : ['Password is required']
	agreedToTermsErrors.value = agreedToTerms.value ? [] : ['You must agree to the terms of use to continue']
	validate(email.value, password.value)
}
</script>
