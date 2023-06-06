<script setup lang="ts">
import { ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from "chart.js/auto";

const weights = ref([]);

const weightChartElement = ref(null);

const weightChart = shallowRef(null);

const weightInput = ref(0);

const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 };
});

const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime(),
  });
};

watch(
  weights,
  newWeights => {
    const weights = [...newWeights];

    if (weightChart.value) {
      weightChart.value.data.labels = weights
        .sort((a, b) => a.date - b.date)
        .map(weight => new Date(weight.date).toLocaleDateString())
        .slice(-7);

      weightChart.value.data.datasets[0].data = weights
        .sort((a, b) => a.date - b.date)
        .map(weight => weight.weight)
        .slice(-7);

      weightChart.value.update();
      return;
    }
    nextTick(() => {
      weightChart.value = new Chart(weightChartElement.value.getContext("2d"), {
        type: "line",
        data: {
          labels: weights
            .sort((a, b) => a.date - b.date)
            .map(weight => new Date(weight.date).toLocaleDateString()),
          datasets: [
            {
              label: "Weight",
              data: weights
                .sort((a, b) => a.date - b.date)
                .map(weight => weight.weight),
              backgroundColor: "rgba(255, 105, 180, 0.2)",
              borderColor: "rgba(255, 105, 180, 1)",
              borderWidth: 1,
              fill: true,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
        },
      });
    });
  },
  { deep: true }
);
</script>

<template>
  <main>
    <div>
      <span> {{ currentWeight.weight }}</span>
      <small> Current weight (lb)</small>
    </div>

    <form @submit.prevent="addWeight">
      <input type="number" step="0.1" v-model="weightInput" />
      <input type="submit" value="Add Weight" />
    </form>

    <div v-if="weights && weights.length > 0">
      <h2>Last 7 days</h2>
      <div>
        <canvas ref="weightChartElement"></canvas>
      </div>

      <div>
        <h2>Weight History</h2>
        <ul>
          <li v-for="weight in weights">
            <span> {{ weight.weight }}lb </span>
            <small>
              {{ new Date(weight.date).toLocaleDateString() }}
            </small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "montserrat", sans-serif;
}

body {
  background-color: #eee;
}

main {
  padding: 1.5rem;
}

h1 {
  font-size: 2em;
  text-align: center;
  margin-bottom: 2rem;
}

h2 {
  margin-bottom: 1rem;
  color: #888;
  font-weight: 400;
}
</style>
