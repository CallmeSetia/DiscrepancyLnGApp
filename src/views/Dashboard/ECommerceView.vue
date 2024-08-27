<script setup>
import DataStatsOne from '@/components/DataStats/DataStatsOne.vue'
import ChartOne from '@/components/Charts/ChartOne.vue'
import ChartThree from '@/components/Charts/ChartThree.vue'
import ChartTwo from '@/components/Charts/ChartTwo.vue'
import ChatCard from '@/components/ChatCard.vue'
import MapOne from '@/components/Maps/MapOne.vue'
import TableOne from '@/components/Tables/TableOne.vue'
import DefaultLayout from '@/layouts/DefaultLayout.vue'
import DefaultCard from "@/components/Forms/DefaultCard.vue";
import {ref} from "vue";
import Papa from 'papaparse';
import ButtonDefault from "@/components/Buttons/ButtonDefault.vue";
import moment from 'moment';
// @ts-ignore
import VueApexCharts from 'vue3-apexcharts'
const file = ref('');
const content = ref([]);
const parsed = ref(false);
const chartData = ref({
  series: [],
  labels: []
});

function handleFileUpload( event ){
  file.value = event.target.files[0];
  parseFile();
}
function  parseFile(){
  Papa.parse( file.value, {
    header: true,
    skipEmptyLines: true,
    complete: function( results ){
      content.value = results;

    }.bind(this)
  } );
}

function handleAnalisaDiskrepansi() {
  // if (!parsed.value || !content.value.data) return;

  const discrepancies = content.value.data.map(row => {
    const FSOKirim = parseFloat(row.FSOKirim) || 0;
    const FSOTerima = parseFloat(row.FSOTerima) || 0;
    const discrepancy = FSOKirim - FSOTerima;

    return {
      Tanggal: row.Tanggal,
      FSOKirim: FSOKirim,
      FSOTerima: FSOTerima,
      Diskrepansi: discrepancy
    };
  });
  content.value.data = discrepancies

  // const RMSE = calculateRMSE(discrepancies);
  // const { UCL, LCL, CL } = calculateUCLLCL(discrepancies, RMSE);
  // controlLimits.value = { UCL, LCL, CL };
  //
  const labels = discrepancies.map(row => row.Tanggal);
  const seriesData = discrepancies.map(row => {
    return [moment(row.Tanggal, 'DD/MM/YYYY').valueOf(), row.Diskrepansi]
  });

  console.log(seriesData)
  chartData.value = {
    series: [
      {
        name: 'Diskrepansi',
        data: seriesData
      }
    ],
    labels: labels
  };
  parsed.value = true;
  console.log(chartData.value.labels);
}

// Calculate RMSE
function calculateRMSE(residuals) {
  const squaredErrors = residuals.map(val => val * val);
  const meanSquaredError = squaredErrors.reduce((acc, val) => acc + val, 0) / squaredErrors.length;
  return Math.sqrt(meanSquaredError);
}

// Calculate UCL, LCL, and CL
function calculateUCLLCL(residuals, RMSE) {
  const mean = residuals.reduce((acc, val) => acc + val, 0) / residuals.length;
  const UCL = mean + 2 * RMSE;
  const LCL = mean - 2 * RMSE;
  return { UCL, LCL, CL: mean };
}

const chart = ref(null)
const apexOptions = {
  legend: {
    show: true,
    position: 'top',
    horizontalAlign: 'left'
  },
  colors: ['#3C50E0'],
  chart: {
    fontFamily: 'Satoshi, sans-serif',
    height: 335,
    type: 'line',
    dropShadow: {
      enabled: true,
      color: '#623CEA14',
      top: 10,
      blur: 4,
      left: 0,
      opacity: 0.1
    },
    toolbar: {
      show: true,
      tools: {
        download: true,
        selection: true,
        zoomin: true,
        zoomout: true,
        pan: true,
        reset: true
      }
    }
  },
  responsive: [
    {
      breakpoint: 1024,
      options: {
        chart: {
          height: 300
        }
      }
    },
    {
      breakpoint: 1366,
      options: {
        chart: {
          height: 350
        }
      }
    }
  ],
  stroke: {
    width: [2],
    curve: 'smooth'
  },
  fill: {
    type: 'gradient',
    gradient: {
      shadeIntensity: 1,
      inverseColors: false,
      opacityFrom: 0.5,
      opacityTo: 0,
      stops: [0, 90, 100]
    }
  },
  grid: {
    xaxis: {
      lines: {
        show: true
      }
    },
    yaxis: {
      lines: {
        show: true
      }
    }
  },
  dataLabels: {
    enabled: false
  },
  markers: {
    size: 0,
    colors: '#fff',
    strokeColors: ['#3056D3'],
    strokeWidth: 3,
    strokeOpacity: 0.9,
    strokeDashArray: 0,
    fillOpacity: 1,
    discrete: [],
    hover: {
      size: undefined,
      sizeOffset: 5
    }
  },
  xaxis: {
    type: 'datetime',
    axisBorder: {
      show: false
    },
    axisTicks: {
      show: false
    },
    labels: {
      rotate: -45,
      formatter: function (value) {
        return new Date(value).toLocaleDateString('id-ID', { year: 'numeric', month: 'short', day: 'numeric' });
      }
    }
  },
  yaxis: {
    title: {
      style: {
        fontSize: '1px'
      }
    }
  },
  tooltip: {
    enabled: true,
    theme: 'light',
    x: {
      show: true,
      format: 'dd MMM yyyy'
    },
    y: {
      show: true,
      formatter: function (val) {
        return `Diskrepansi: ${val}`;
      }
    }
  }
};


</script>

<template>
  <DefaultLayout>
    <div class="grid grid-cols-1 gap-1 md:grid-cols-1 md:gap-6 xl:grid-cols-1 2xl:gap-7.5">
      <DefaultCard cardTitle="File upload">
        <div class="flex flex-col gap-5.5 p-6.5">
          <div>
            <label class="mb-3 block text-sm font-medium text-black dark:text-white">
              Attach file
            </label>
            <input
                type="file"
                accept=".csv" @change="handleFileUpload( $event )"
                class="w-full cursor-pointer rounded-lg border-[1.5px] border-stroke bg-transparent font-medium outline-none transition file:mr-5 file:border-collapse file:cursor-pointer file:border-0 file:border-r file:border-solid file:border-stroke file:bg-whiter file:py-3 file:px-5 file:hover:bg-primary file:hover:bg-opacity-10 focus:border-primary active:border-primary disabled:cursor-default disabled:bg-whiter dark:border-form-strokedark dark:bg-form-input dark:file:border-form-strokedark dark:file:bg-white/30 dark:file:text-white dark:focus:border-primary"
            />
          </div>
          <ButtonDefault route="/" label="Analisa Dikrepansi" customClasses="bg-primary text-white rounded-md" @click="handleAnalisaDiskrepansi" />
        </div>

      </DefaultCard>
    </div>
    <div v-if="parsed" class="mt-4 grid grid-cols-1 gap-1 md:mt-6 md:gap-6 2xl:mt-7.5 2xl:gap-7.5">
      <!-- ====== Chart One Start -->
      <div
          class="col-span-12 rounded-sm border border-stroke bg-white px-5 pt-7.5 pb-5 shadow-default dark:border-strokedark dark:bg-boxdark sm:px-7.5 xl:col-span-8"
      >
        <div class="flex flex-wrap items-start justify-between gap-3 sm:flex-nowrap">
          <div class="flex w-full flex-wrap gap-3 sm:gap-5">
            <div class="flex min-w-47.5">
          <span
              class="mt-1 mr-2 flex h-4 w-full max-w-4 items-center justify-center rounded-full border border-primary"
          >
            <span class="block h-2.5 w-full max-w-2.5 rounded-full bg-primary"></span>
          </span>
              <div class="w-full">
                <p class="font-semibold text-primary">Diskrepansi</p>
                <p class="text-sm font-medium"></p>
              </div>
            </div>
            <div class="flex min-w-47.5">
<!--          <span-->
<!--              class="mt-1 mr-2 flex h-4 w-full max-w-4 items-center justify-center rounded-full border border-secondary"-->
<!--          >-->
<!--            <span class="block h-2.5 w-full max-w-2.5 rounded-full bg-secondary"></span>-->
<!--          </span>-->
<!--              <div class="w-full">-->
<!--                <p class="font-semibold text-secondary">Total Sales</p>-->
<!--                <p class="text-sm font-medium">12.04.2022 - 12.05.2022</p>-->
<!--              </div>-->
            </div>
          </div>
          <div class="flex w-full max-w-45 justify-end">
            <div class="inline-flex items-center rounded-md bg-whiter p-1.5 dark:bg-meta-4">
              <button
                  class="rounded bg-white py-1 px-3 text-xs font-medium text-black shadow-card hover:bg-white hover:shadow-card dark:bg-boxdark dark:text-white dark:hover:bg-boxdark"
              >
                Yearly
              </button>
              <button
                  class="rounded py-1 px-3 text-xs font-medium text-black hover:bg-white hover:shadow-card dark:text-white dark:hover:bg-boxdark"
              >
                Month
              </button>
              <button
                  class="rounded py-1 px-3 text-xs font-medium text-black hover:bg-white hover:shadow-card dark:text-white dark:hover:bg-boxdark"
              >
                Week
              </button>
            </div>
          </div>
        </div>
        <div>
          <div id="chartOne" class="-ml-5">
            <VueApexCharts
                type="area"
                height="350"
                :options="apexOptions"
                :series="chartData.series"
                ref="chart"
            />
          </div>
        </div>
      </div>
      <!-- ====== Chart One End -->

      <!-- ====== Chart Two Start -->
<!--      <ChartTwo />-->
      <!-- ====== Chart Two End -->

      <!-- ====== Chart Three Start -->
<!--      <ChartThree />-->
      <!-- ====== Chart Three End -->

      <!-- ====== Map One Start -->
<!--      <MapOne />-->
      <!-- ====== Map One End -->

      <!-- ====== Table One Start -->
      <div class="col-span-12 xl:col-span-8">
        <TableOne />
      </div>
      <!-- ====== Table One End -->

      <!-- ====== Chat Card Start -->
<!--      <ChatCard />-->
      <!-- ====== Chat Card End -->
    </div>
  </DefaultLayout>
</template>
