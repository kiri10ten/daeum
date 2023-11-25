<template>

     <div class="flex  justify-end  pr-10 pt-4 ">
     <UButton color="white" @click="logout"> log out </UButton>
     </div>
    <div class="  flex flex-col p-28 w-full items-center text">
        
        <h1 class="  text-xl p-10">Create New Habit</h1>
        
        <div class = "flex flex-row gap-5 p-4" >
            <UTextarea :rows="1"  v-model="habitName" placeholder="New Habit..." />
             <UButton @click="addHabit"> Add the Habit</UButton>
         </div>

      <div class="flex flex-row">
        <habitlog  :updateHabits="fetchHabits" />
    <habitintense/>
      </div>


     
       
    </div>
</template>

<script setup>
const habitName = ref('');
const client = useSupabaseClient();

const user = useSupabaseUser()
const addHabit = async () => {

    if (habitName.value != ''){
        
    try {
        await client.from('HABITS').insert([{ HABIT_NAME: habitName.value, USER_ID: user.value.id }]);
        habitName.value = '';
        console.log(user.value.id)
        alert('Habit added successfully');

        fetchHabits();
        
 
    } catch (error) {
        console.error(error);
    }
    }
    else {
        alert("please enter some Habits pwease 🥹")
    };



}
const logout = async () => {
    await client.auth.signOut()
    navigateTo('/login')
}



</script>
