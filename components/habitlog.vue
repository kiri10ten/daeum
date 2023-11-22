<template>
    <div>
        <h2>Today's Habits</h2>
        <ul>
            <li v-for="habit in unloggedHabits" :key="habit.id">
                <input type="checkbox" v-model="habit.done" />
                {{ habit.HABIT_NAME }}
            </li>
        </ul>

        <button @click="addToHabitLog">Add to habit log</button>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const client = useSupabaseClient();
const habits = ref([]);
const unloggedHabits = ref([]);

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
    } catch (error) {
        console.error(`Error adding habit ${habit.HABIT_ID} to log:`, error);
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
