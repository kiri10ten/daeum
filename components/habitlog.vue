

<template>
    <div class="p-4 border-2 border-slate-600 ">
        <h2 class="pb-4">Today's Habits</h2>
        <ul>
            <li class="" v-for="habit in unloggedHabits" :key="habit.id">
               
                <UCheckbox  v-model="habit.done" :model-value="false" >   
                <template #label> 

                  <span> {{ habit.HABIT_NAME }}</span>
                </template>
             </UCheckbox> 
            </li>
            
        </ul>

    <div class="pt-2">
    <UButton  @click="addToHabitLog"> Add to habit log</UButton>
    </div>
       
    </div>
</template>

<script setup>


const client = useSupabaseClient();
const habits = ref([]);
const unloggedHabits = ref([]);
const totalHabitsLogged = ref(0); // Initialize totalHabitsLogged with 0

const fetchHabits = async () => {
    try {
        const currentDate = new Date().toISOString().split('T')[0];
        const response = await client.from('HABITS').select('*');
        const allHabits = response.data;

        // Filter out habits that are already logged for the current date
        const loggedHabits = await client
            .from('HABIT_LOG')
            .select('HABIT_ID')
            .eq('DATE', currentDate);
        unloggedHabits.value = allHabits.filter(
            (habit) => !loggedHabits.data.some((log) => log.HABIT_ID === habit.HABIT_ID)
        );

        // Recalculate totalHabitsLogged based on the fetched data
        totalHabitsLogged.value = loggedHabits.data ? loggedHabits.data.length : 0;
    } catch (error) {
        console.error('Error fetching habits:', error);
    }
};

const addToHabitLog = async () => {
    try {
        const habitsToAdd = unloggedHabits.value.filter((habit) => habit.done);
        for (const habit of habitsToAdd) {
            await addHabitToLog(habit);
        }
        alert(" You have done it ")
        // Refresh habits after adding to the log
        await fetchHabits();
    } catch (error) {
        console.error('Error adding habits to log:', error);
    }
};

const addHabitToLog = async (habit) => {
    try {
        if (await habitExists(habit.HABIT_ID)) {
            // Check if a log entry already exists for the current date and habit
            const currentDate = new Date().toISOString().split('T')[0];
            const existingLogEntry = await client
                .from('HABIT_LOG')
                .select('HABIT_ID')
                .eq('HABIT_ID', habit.HABIT_ID)
                .eq('DATE', currentDate);

            if (!existingLogEntry.data.length) {
                // Insert a new log entry if it doesn't already exist
                await client.from('HABIT_LOG').insert({
                    HABIT_ID: habit.HABIT_ID,
                    LOG: habit.done,
                    DATE: currentDate,
                });
            } else {
                console.warn(`Habit with ID ${habit.HABIT_ID} already logged for today. Skipping.`);
            }
        } else {
            console.warn(`Habit with ID ${habit.HABIT_ID} does not exist. Skipping.`);
        }

        await updateHabitIntensity();
    } catch (error) {
        console.error(`Error adding habit ${habit.HABIT_ID} to log:`, error);
    }
};

const updateHabitIntensity = async () => {
    try {
        const currentDate = new Date().toISOString().split('T')[0];
        const habitLogs = await client
            .from('HABIT_LOG')
            .select('LOG_ID') // Select any field, as we only want to check the number of records
            .eq('DATE', currentDate);

        console.log('habitLogs:', habitLogs);
        console.log(`Request URL: ${client.from('HABIT_LOG').select('LOG_ID').eq('DATE', currentDate).url}`);

        // Update habit intensity table
        await client.from('HABIT_INTENSITY').upsert([
            {
                TOTAL: habitLogs.data ? habitLogs.data.length : 0,
                DATE: currentDate,
            },
        ]);
    } catch (error) {
        console.error('Error updating habit intensity:', error);
    }
};

const habitExists = async (habitId) => {
    try {
        const response = await client
            .from('HABITS')
            .select('HABIT_ID')
            .eq('HABIT_ID', habitId);
        return response.data.length > 0;
    } catch (error) {
        console.error(`Error checking if habit with ID ${habitId} exists:`, error);
        return false;
    }
};

onMounted(fetchHabits);
</script>