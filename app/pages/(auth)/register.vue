<script lang="ts" setup>
const router = useRouter()
const state = reactive({
  name: undefined,  
  email: undefined,
  password: undefined,
  value: undefined
})

const radio = [
    {
        label: 'Wedder', value: 'wedder'
    },
    {
        label: 'Visitor', value: 'visitor'
    }
]
const value = ref('Wedder')

const loading = ref(false)
const error = ref('')

async function handleSubmit(){
  loading.value = true 
  error.value = ''

  try{
    const response = await $fetch('/auth/register',{
      baseURL: useRuntimeConfig().public.apiBase,
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
      },
      body: {
        name: state.name,
        email: state.email,
        password: state.password,
        role: value.value
      }
      
    }) as { token: string; data: any }
    localStorage.setItem('token', response.token)
    localStorage.setItem('user', JSON.stringify(response.data))
    await router.push('/')
  } catch (err: any) {
    error.value = err?.data?.errors;
  } finally {
    loading.value = false
  }
}

</script>
<template>

<div 
  v-if="error" 
  class="bg-red-500/10 border border-red-500/20 text-red-400 px-4 py-3 rounded-lg text-sm flex items-center gap-2"
>
  <span class="font-medium">Error:</span>
  <span>{{ error }}</span>
</div>
  
  <div class="flex justify-center items-center min-h-screen">
    <UForm class="flex flex-col gap-4 p-6 rounded shadow-md" 
      @submit.prevent="handleSubmit">
      <div class="text-2xl font-bold mb-4">Register</div>

        <UFormField label="Name" name="Name">
        <UInput class="w-full" v-model="state.name" />
      </UFormField>
      <UFormField label="Email" name="Email">
        <UInput class="w-full" v-model="state.email" />
      </UFormField>

      <UFormField label="Password" name="Password">
        <UInput class="w-full" v-model="state.password" />
      </UFormField>
      <URadioGroup v-model="value" :items="radio"></URadioGroup>
      <UButton type="submit" :loading="loading" class="flex items-center justify-center">Submit</UButton>

<div class="text-white">Have an account ? 
  <NuxtLink class="underline" to="/">Log In
  </NuxtLink ></div>
    </UForm>
  </div>
</template>