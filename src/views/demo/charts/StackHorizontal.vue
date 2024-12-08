<template>
  <div>
    <button @click="updateChartData">Update Data</button>
    <div ref="chartRef" style="width: 100%; height: 500px"></div>
  </div>
</template>

<script setup lang="ts">
  import { useECharts } from '@/hooks/web/useECharts';
  import { onMounted, ref } from 'vue';

  // Ref for the chart container
  const chartRef = ref<any>(null);
  const { setOptions } = useECharts(chartRef);

  // Initial data
  const initialData = ref({
    CL: [20, 30, 40, 10, 45],
    KN: [10, 30, 20, 50, 25],
    CNT: [70, 40, 40, 40, 30],
  });

  onMounted(() => {
    // Set initial chart options
    setOptions({
      legend: {
        icon: 'circle',
      },
      xAxis: {
        type: 'value',
        axisLabel: {
          formatter(value, index) {
            if (index === 0) return value.toString();
            return value + '%';
          },
        },
      },
      yAxis: {
        type: 'category',
        data: ['2020', '2021', '2022', '2023', '2024'],
      },
      series: [
        {
          name: 'CL',
          type: 'bar',
          stack: 'total',
          label: {
            show: true,
          },
          emphasis: {
            focus: 'series',
          },
          data: initialData.value.CL,
        },
        {
          name: 'KN',
          type: 'bar',
          stack: 'total',
          label: {
            show: true,
          },
          emphasis: {
            focus: 'series',
          },
          data: initialData.value.KN,
        },
        {
          name: 'CNT',
          type: 'bar',
          stack: 'total',
          label: {
            show: true,
          },
          emphasis: {
            focus: 'series',
          },
          data: initialData.value.CNT,
        },
      ],
    });
  });

  // Method to update chart data
  function updateChartData() {
    // Generate new random data for each series
    const newData = {
      CL: Array.from({ length: 5 }, () => Math.floor(Math.random() * 50)),
      KN: Array.from({ length: 5 }, () => Math.floor(Math.random() * 50)),
      CNT: Array.from({ length: 5 }, () => Math.floor(Math.random() * 50)),
    };

    // Update the chart options with the new data
    setOptions(
      {
        series: [
          { name: 'CL', data: newData.CL },
          { name: 'KN', data: newData.KN },
          { name: 'CNT', data: newData.CNT },
        ],
      },
      false,
    );

    // Update the reactive `initialData` for state consistency
    initialData.value = newData;
  }
</script>
