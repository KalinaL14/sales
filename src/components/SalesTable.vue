<template>
  <div class="sales-table">
    <table>
      <thead>
        <tr>
          <th class="col-title">Показатель</th>
          <th>Сегодня</th>
          <th>Вчера</th>
          <th>Этот день недели</th>
        </tr>
      </thead>
      <tbody>
        <template v-for="(item, index) in data" :key="index">
          <tr @click="toggle(index)">
            <td class="col-title fixed-title">{{ item.title }}</td>
            <td class="col-today">{{ format(item.today) }}</td>
            <td :class="['col-change', getChangeClass(item.change, index)]">
              {{ format(item.yesterday) }}
              <span class="change-inline" :class="getTextChangeClass(item.change)">
                ({{ item.change > 0 ? '+' : '' }}{{ item.change }}%)
              </span>
            </td>
            <td :class="['col-week', getWeekColor(index, item.title)]">
              {{ format(item.week) }}
            </td>
          </tr>
          <tr v-if="expandedIndex === index" class="chart-row">
            <td colspan="4">
              <apexchart height="220" type="line" :options="chartOptions(item.title)" :series="chartSeries(item)" />
              <table v-if="item.children" class="nested-table">
                <tr v-for="(child, cidx) in item.children" :key="cidx">
                  <td class="col-title fixed-title nested-title">
                    {{ child.title }}
                  </td>
                  <td class="col-today">{{ format(child.today) }}</td>
                  <td :class="['col-change', getChangeClass(child.change, cidx)]">
                    {{ format(child.yesterday) }}
                    <span class="change-inline" :class="getTextChangeClass(child.change)">
                      ({{ child.change > 0 ? '+' : '' }}{{ child.change }}%)
                    </span>
                  </td>
                  <td :class="['col-week', getWeekColor(cidx, child.title)]">
                    {{ format(child.week) }}
                  </td>
                </tr>
              </table>
            </td>
          </tr>
        </template>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import ApexChart from 'vue3-apexcharts';
import { salesData as data } from '../mockData.js';

const expandedIndex = ref(null);
const toggle = (index) => {
  expandedIndex.value = expandedIndex.value === index ? null : index;
};

const format = (val) => typeof val === 'number' ? new Intl.NumberFormat('ru-RU').format(val) : val;

const getTextChangeClass = (val) => {
  if (val > 0) return 'positive';
  if (val < 0) return 'negative';
  return 'neutral';
};

const getChangeClass = (val, idx) => {
  if (val > 0) return 'cell-green';
  if (val < 0) return 'cell-red';
  return 'cell-neutral';
};

const getWeekColor = (idx, title) => {
  if (title === 'Выручка, руб') return 'cell-red';
  return [2, 3, 4, 5, 10].includes(idx) ? 'cell-neutral' : 'cell-green';
};;

const chartOptions = (title) => ({
  chart: {
    id: title,
    toolbar: { show: false },
    zoom: { enabled: false },
  },
  markers: {
    size: 5,
    strokeWidth: 2,
    hover: { sizeOffset: 3 },
  },
  stroke: {
    curve: 'smooth',
    width: 3,
    colors: ['#28a745'],
  },
  xaxis: {
    categories: ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс'],
    labels: { style: { colors: '#5C4033' } },
    axisBorder: { color: '#5C4033' },
    axisTicks: { color: '#5C4033' },
  },
  yaxis: {
    labels: { style: { colors: '#5C4033' } },
  },
  colors: ['#28a745'],
});

const chartSeries = (item) => [
  {
    name: item.title,
    data: item.chartData || [0, 0, 0, 0, 0, 0, 0],
  },
];
</script>

<style scoped>
.sales-table {
  max-width: 1000px;
  margin: 0 auto;
  font-family: sans-serif;
}

.fixed-title {
  width: 220px;
  min-width: 220px;
  max-width: 220px;
}

.nested-title {
  padding-left: 28px;
}

table {
  width: 100%;
  border-collapse: collapse;
  background: #fff;
}

thead th {
  background: #f3f3f3;
  font-weight: bold;
  border: 2px solid white;
  padding: 16px;
  text-align: left;
}

td {
  border: 2px solid white;
  padding: 16px;
  font-size: 15px;
  vertical-align: middle;
}

.col-title {
  background-color: #f8f0f2;
  font-weight: 500;
}

.col-today {
  background-color: #e7f3fb;
}

.col-week {
  font-weight: 500;
}

.cell-green {
  background-color: #e6f4ec;
}

.cell-red {
  background-color: #f2d4da;
}

.cell-neutral {
  background-color: #f8f0f2;
}

.change-inline {
  margin-left: 6px;
  font-size: 13px;
  font-weight: bold;
}

.change-inline.positive {
  color: #28a745;
}

.change-inline.negative {
  color: #dc3545;
}

.change-inline.neutral {
  color: #666;
}

tr:hover {
  background: #f9f9f9;
  cursor: pointer;
}

.chart-row {
  background: #fcfcfc;
}

.nested-table {
  margin-top: 1rem;
  border-collapse: collapse;
  width: 100%;
}

.nested-table td {
  padding: 16px;
  border: 2px solid white;
  font-size: 14px;
}
</style>
