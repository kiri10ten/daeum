<script setup>
import { ref } from 'vue'


const client = useSupabaseClient()
const email = ref('')
const password = ref('')

const handleSignin = async () => {
    // Try signing in the user using passwordless authentication
    try {
        const { error }= await client.auth.signInWithPassword({ email: email.value, password: password.value })
        if (error) throw error;
        router.push("index")
    } catch (error) {
        // Handle error
        console.error(error)
    }
}
</script>

<template>
    <div class="p-25 flex flex-col w-full h-full">
    <form @submit.prevent="handleSignin">
        <input type="email" v-model="email" />
        <input type="password" v-model="password" />
        <button type="submit">Sign In</button>
    </form>
    </div>
</template>