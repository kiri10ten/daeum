<script setup lang="ts">
const user = useSupabaseUser()
const { auth } = useSupabaseClient()

const redirectTo = `${useRuntimeConfig().public.baseUrl}/confirm`

watchEffect(() => {
    if (user.value) {
        navigateTo('/confirm')
    }
})
</script>

<template>
    <div class="min-h-full flex flex-col justify-center pt-52 sm:px-6 lg:px-8">
        <h2 class="my-6 text-center text-xl  u-text-white">
            Sign in to  log your Habits :)
        </h2>
        <div class=" flex justify-center">
            <UButton class="mt-3   w-44" color="black" label="Github" 
                @click="auth.signInWithOAuth({ provider: 'github', options: { redirectTo } })" />
        </div>
    </div>
</template>
