<script setup>

import CalHeatmap from 'cal-heatmap';

import 'cal-heatmap/cal-heatmap.css';

const client = useSupabaseClient();
const cal = new CalHeatmap();
const transformedData = ref([]);

const fetchIntensity = async () => {
    try {
        const response = await client.from('HABIT_INTENSITY').select('*');
        const allintensity = response.data;
        console.log(allintensity);
        transformedData.value = allintensity.map(item => ({
            date: item.DATE,
            intensity: item.TOTAL,
        }));
        console.log(transformedData.value);

        // Call cal.paint only after fetching the data
        cal.paint({
            data: {
                source: transformedData.value,
                x: 'date',
                y: 'intensity',
            },
            verticalOrientation: true,
            range: 1,
            date: {
                start: new Date('2023-11-23')
            },
            scale: { color: { type: 'diverging', scheme: 'PRGn', domain: [-10, 15] } },
            domain: {
                type: 'month',
                padding: [10, 10, 10, 10],
                label: { position: 'top' },
            },
            subDomain: { type: 'xDay', radius: 2, width: 15, height: 15, label: 'D' },
        });

    } catch (error) {
        console.error('Error fetching habits:', error);
    }
};

onMounted(() => {
    fetchIntensity();
});

</script>

<template>
    <div id="cal-heatmap"></div>
</template>