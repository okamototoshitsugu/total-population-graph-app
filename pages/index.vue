<template>
<div>
    <h1>
      都道府県別<br>総人口推移グラフ
    </h1>
    <form>
      <label>都道府県</label>
      <b-form-checkbox-group v-model="checkedPrefCodes">
        <b-form-checkbox
          v-for="prefecture in prefectures"
          :key="prefecture.prefCode"
          :value="prefecture.prefCode"
          class=""
        >
          {{ prefecture.prefName }}
        </b-form-checkbox>
      </b-form-checkbox-group>
    </form>
    <highcharts :options="chartOptions"></highcharts>
</div>
</template>

<script>
import { Chart } from 'highcharts-vue'
import Highcharts from 'highcharts'



const API_KEY = 'IGLT4AfOzzGG9vf6RtNJGLax9xPAE0ceW0hgTwHT'



export default {
  components: {
    highcharts: Chart
  },
  data() {
    return {
      prefectures: [],
      checkedPrefCodes: null,
      chartOptions: {
        title: {
          text: null
        },
        tooltip: {
          formatter: function() {
            return (
              '<b>' +
              this.x +
              '</b><br/>' +
              this.series.name +
              ': ' +
              Highcharts.numberFormat(this.y, 0, ',', ',') + '人'
            )
          }
        },
        xAxis: {
          title: {
            text: '年度'
          }
        },
        yAxis: {
          title: {
            text: '人口数'
          },
          labels: {
            formatter: function() {
              return Highcharts.numberFormat(this.value, 0, ',', ',')
            }
          }
        },

        plotOptions: {
          series: {
            pointStart: 1980,
            pointInterval: 3
          }
        },
        series: []
      }
    }
  },
  watch: {
    checkedPrefCodes: async function(checkedPrefCodes) {
      if (checkedPrefCodes == null) return
      const series = []
      for (const checkedPrefCode of checkedPrefCodes) {
        const data = await this.$axios
          .get(
            'https://opendata.resas-portal.go.jp/api/v1/population/composition/perYear?prefCode=' + checkedPrefCode,
            {
              headers: {
                'Content-Type': 'application/json',
                'X-API-KEY': API_KEY
              }
            }
          )
          .then(response => response.data.result.data[0].data)
        series.push({
          name: this.getPrefNameByCode(checkedPrefCode),
          data: data.map(d => d.value)
        })
      }
      this.chartOptions.series = series
    }
  },
  async mounted() {
    this.prefectures = await this.$axios
      .get('https://opendata.resas-portal.go.jp/api/v1/prefectures', {
        headers: { 
          'Content-Type': 'application/json',
           'X-API-KEY': API_KEY
        }
      })
      .then(response => response.data.result)
  },
  methods: {
    getPrefNameByCode(code) {
      return this.prefectures.filter(
        prefecture => prefecture.prefCode === code
      )[0].prefName
    }
  }
}
</script>


<style>

</style>
