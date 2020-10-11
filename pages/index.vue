<template>
  <div class="container">
    <p>
      <label v-for="item in prefecturesList" :key="item.prefCode">
        <input
          v-model="prefecturesCheckedItems"
          type="checkbox"
          :name="item.prefName"
          :value="item.prefCode"
          @change="onChange"
        />
        <span>{{ item.prefName }}</span>
      </label>
    </p>
    <div>
      <input id="scales" type="checkbox" name="scales" checked />
      <label for="scales">Scales</label>
    </div>
    <highcharts :options="hogeChartOptions"></highcharts>
  </div>
</template>

<script lang="ts">
import {
  defineComponent,
  useFetch,
  reactive,
  useContext,
  toRefs,
} from 'nuxt-composition-api'
import { Chart } from 'highcharts-vue'

export default defineComponent({
  components: {
    highcharts: Chart,
  },
  setup(_props) {
    const { state, onChange } = useHighCharts()

    return { ...toRefs(state), onChange }
  },
})

type HogeChartOptions = {
  title: { text: string }
  xAxis: { categories: number[]; crosshair: boolean }
  yAxis: { title: { text: string }; labels: { format: string } }
  series: {}[]
}

type State = {
  prefecturesList: []
  prefecturesCheckedItems: any[]
  hogeChartOptions: HogeChartOptions
}

type resData = {
  name: string
  data: { y: number; name: number }
}

const useHighCharts = () => {
  const { app } = useContext()

  const state: State = reactive({
    prefecturesList: [],
    prefecturesCheckedItems: [],

    hogeChartOptions: {
      title: {
        text: ' 都道府県別の総人口推移グラフ',
      },
      xAxis: {
        categories: [
          1960,
          1965,
          1970,
          1975,
          1980,
          1985,
          1990,
          1995,
          2000,
          2005,
          2010,
          2015,
          2020,
          2025,
          2030,
          2035,
          2040,
          2045,
        ],
        crosshair: true, // マウスを当てると線が出て、数値が分かりやすくなる
      },
      yAxis: {
        title: {
          text: '人口(千人)',
        },
        labels: {
          format: '{value}', // y軸の目盛り幅が値によって動的に変わる
        },
      },
      series: [],
    },
  })

  useFetch(async () => {
    const res = await app.$axios.get('api/v1/prefectures', {
      method: 'GET',
      headers: {
        'X-API-KEY': 'ujqwj0KFo37MEjxr1OTyYWkvBp5c8MJxC99SsquU',
      },
    })

    state.prefecturesList = res.data.result
  })

  const onChange = async () => {
    state.hogeChartOptions.series = []
    for (const item of state.prefecturesCheckedItems) {
      const res = await app.$axios.get(
        `api/v1/population/composition/perYear?prefCode=${item}`,
        {
          method: 'GET',
          headers: {
            'X-API-KEY': 'ujqwj0KFo37MEjxr1OTyYWkvBp5c8MJxC99SsquU',
          },
        }
      )
      const totalPopulationData = res.data.result.data[0]

      const correctedData = totalPopulationData.data.map(
        ({ year, value }: { year: number; value: number }) => {
          return {
            name: year,
            y: Number(String(value).slice(0, -4)),
          }
        }
      )

      const resData = {
        name: totalPopulationData.label,
        data: correctedData,
      }

      state.hogeChartOptions.series.push(resData)
    }
  }

  return { state, onChange }
}
</script>

<style lang="scss" scoped></style>
