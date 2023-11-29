<template>
    <div class="flex justify-end pr-10 pt-4">
        <UButton color="white" @click="logout">Log Out</UButton>
       
    </div>
    <div class="flex flex-col p-28 w-full items-center text">
        <h1 class="text-xl p-10">Create New Habit</h1>
        <div class="flex flex-row gap-5 p-4">
            <UTextarea :rows="1" v-model="habitName" placeholder="New Habit..." />
            <UButton @click="addHabit">Add the Habit</UButton>
        </div>
        <div class="flex flex-row">
            <habitlog :rerender="rerender"/>
            <habitintense />
        </div>
    </div>
</template>

<script setup>

const rerender = ref(false);
const habitName = ref('');
const client = useSupabaseClient();
const user = useSupabaseUser();
// Custom event emitter
const addHabit = async () => {
    const trimmedHabitName = habitName.value.trim();
    if (trimmedHabitName !== '') {
        try {
            await client.from('HABITS').insert([{ HABIT_NAME: trimmedHabitName, USER_ID: user.value.id }]);
            habitName.value = '';
            alert('Habit added successfully');
            rerender.value = !rerender.value;
            

        } catch (error) {
            console.error(error);
        }
    } else {
        alert("Please enter some habits, please 🥹");
    }
};

const logout = async () => {
    await client.auth.signOut();
    navigateTo('/login');
};
</script>