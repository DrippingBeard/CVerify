<template>
  <div class="medium-editor">
    <div class="row row-equal">
      <div class="row">
        <div class="flex xs12 xl3 md12">
          <va-card class="flex">
            <va-card-content>
              <div class="row row-separated">
                <div class="flex xs12">
                  <h2 class="va-h2 ma-5 va-text-center">
                    <va-avatar size="150px">
                      <!-- <i class="fas fa-user-alt" style="font-size: 100px"> </i> -->
                      <img :src="gh.avatar_url" />
                    </va-avatar>
                  </h2>
                  <p class="va-text-center"></p>
                </div>
                <div class="flex">
                  <va-card-title>Basic Details</va-card-title>
                  <va-card-content>Name: {{ candidate.name }}</va-card-content>
                  <va-card-content>Email: {{ candidate.email }}</va-card-content>
                  <va-card-content>Github: {{ candidate.github }}</va-card-content>
                  <va-card-content>LinkedIn:{{ candidate.linkedin }}</va-card-content>
                </div>
              </div>
            </va-card-content>
          </va-card>
        </div>
        <div class="flex md9 xs12" v-if= "found">
          <!-- Fetch data from Github API-->
          <!-- <va-data-table :items="repos" :columns="columns" /> -->
          <img
            :src="`http://github-profile-summary-cards.vercel.app/api/cards/stats?username=${candidate.github}&theme=github`"
          />

          <img
            :src="`http://github-profile-summary-cards.vercel.app/api/cards/repos-per-language?username=${candidate.github}&theme=github&exclude={exclude}`"
          />

          <img
            :src="`http://github-profile-summary-cards.vercel.app/api/cards/most-commit-language?username=${candidate.github}&theme=github&exclude={exclude}`"
          />

          <img
            :src="`https://github-profile-summary-cards.vercel.app/api/cards/profile-details?username=${candidate.github}&theme=github`"
            style="width: 90%"
          />
        </div>
        <div class="flex md9 xs12 items-center" v-else>
          <!-- Fetch data from Github API-->
          <!-- <va-data-table :items="repos" :columns="columns" /> -->
          <va-card class="flex text-center">
            <va-card-content class="">
              <i class="fas fa-exclamation-triangle" style="font-size: 48px; color: gray"></i>
              <p>Github Username not found</p>
            </va-card-content>
          </va-card>
        </div>
      </div>

      <div class="flex">
        <va-card class="row row-separated">
          <va-card-title>Skills</va-card-title>
          <va-card-content>
            <div class="flex xs12">
              <va-chip-group>
                <va-chip v-for="skill in candidate.skills" :key="skill" color="primary" style="margin: 8px">
                  {{ skill }}
                </va-chip>
              </va-chip-group>
            </div>
          </va-card-content>
        </va-card>
      </div>
    </div>
    <br />
    <div class="flex items-center">
      <va-card>
        <va-card-title>Actions </va-card-title>
        <va-card-content>
          <va-button icon="arrow_forward" icon-color="#fff" class="flex" style="margin: 8px" @click="accept()">
            Accept
          </va-button>
          <va-button icon="clear" color="danger" class="flex" style="margin: 8px" @click="reject()"> Reject </va-button>
        </va-card-content>
      </va-card>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useRouter } from 'vue-router'
import { ref, onMounted } from 'vue'
import { useToast } from 'vuestic-ui'
import axios from 'axios'

const { init: toast } = useToast()

const router = useRouter()
const candidate = ref({
  id: null,
  name: '',
  email: '',
  skills: [],
  education: [],
  github: '',
  linkedin: '',
  urls: [],
  status: '',
})
const gh = ref({
  login: '',
  avatar_url: '',
  public_repos: '',
})
interface Repos {
  name: string
  url: string
  forks: number
}
const serverUrl =
  import.meta.env.VITE_CVERIFY_SERVER_URL === '' ? 'http://localhost:6969' : import.meta.env.VITE_CVERIFY_SERVER_URL
const repos = ref<Repos[]>([])
const found = ref()

onMounted(async () => {
  const id = router.currentRoute.value.params.id
  const endPointUrl = serverUrl + `/candidate/${id}`
  try {
    found.value = true
    const response = await axios.get(endPointUrl)
    candidate.value = response.data
    const name = candidate.value.github
    const res = await axios.get(`https://api.github.com/users/${name}`)
    gh.value = res.data
    const rep = await axios.get(`https://api.github.com/users/${name}/repos`)
    repos.value = rep.data
  } catch (error) {
    found.value = false
    toast({ message: 'Failed to fetch data from server', color: 'danger' })
  }
})
const accept = async () => {
  const id = router.currentRoute.value.params.id
  const endPointUrl = serverUrl + `/candidate/${id}/true`
  try {
    const response = await axios.get(endPointUrl)
    router.push('/dashboard')
  } catch (err) {
    toast({ message: 'Failed to update data on server. Try again later', color: 'danger' })
  }
}
const reject = async () => {
  const id = router.currentRoute.value.params.id
  const endPointUrl = serverUrl + `/candidate/${id}/false`
  try {
    const response = await axios.get(endPointUrl)
    router.push('/dashboard')
  } catch (err) {
    toast({ message: 'Failed to update data on server. Try again later', color: 'danger' })
  }
}
</script>
<style></style>
