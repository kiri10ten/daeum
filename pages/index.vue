<template>
    <div>
        <h1>Create New Habit</h1>
        <label>New habit</label>
        <input v-model="habitName" type="text">
        <button @click="addHabit">Add the habit</button>

        <habitlog />
    </div>
</template>

<script setup>
const habitName = ref('');
const client = useSupabaseClient();


const addHabit = async () => {
    try {
        await client.from('HABITS').insert([{ HABIT_NAME: habitName.value }]);
        habitName.value = '';
        alert('Habit added successfully');

        
 
    } catch (error) {
        console.error(error);
    }
};


</script>
