<template>
  <div id="TotalNetWin">
    <v-card>
      <v-card-text>
        <v-container fluid>
          <v-layout row wrap>
            <v-flex xs9>
              <v-slider class="pa-0" label="size" :max="4000000" :min="1" v-model="size" @input="change" :disabled="network"></v-slider>
            </v-flex>
            <v-flex xs3 class="pl-4">
              <v-text-field class="pa-0" :max="4000000" :min="1" v-model="size" type="number" @change="change" :disabled="network"></v-text-field>
            </v-flex>
          </v-layout>
        </v-container>
      </v-card-text>
    </v-card>
    <br>
    <v-card>
      <v-card-text>
        <p v-if="error !== ''" class="ma-0">{{ error }}</p>
        <v-layout v-else-if="distribution.length !== 0" row wrap>
          <v-flex xs10>
            <v-card>
              <v-card-text>
                <high-chart :options="defaultOption(distribution, categories)" style="display: flex"></high-chart>
              </v-card-text>
            </v-card>
          </v-flex>
          <v-flex xs2 class="pl-3">
            <v-card>
              <v-card-text>
                <tables :options="tableData"></tables>
              </v-card-text>
            </v-card>
          </v-flex>
        </v-layout>
        <v-progress-circular indeterminate class="primary--text" v-else :size="50" style="width:100%;"></v-progress-circular>
      </v-card-text>
    </v-card>
  </div>
</template>

<script>
import bigNumber from 'bignumber.js'

import api from '../../store/api'
import _ from 'lodash'
export default {
  props: ['type', 'title'],
  data () {
    return {
      tableData: {},
      distribution: [],
      categories: [],
      size: 4000000,
      min: 0,
      max: 0,
      error: '',
      network: false
    }
  },
  beforeMount () {
    this.start()
    this.$store.watch(this.$store.getters.getChoose, function (n) {
      this.start()
    }.bind(this))
  },
  watch: {
    '$route': 'start'
  },
  methods: {
    start () {
      let self = this
      self.distribution = []
      self.error = ''
      self.network = true
      api.getDistribution(localStorage.getItem('token'), self.$store.state.projectId.id, self.size, '', this.type).then(res => {
        self.distribution = self.conertData(res.data.chartData)
        self.tableData    = self.defaultTable(res.data.tableData)
        self.network      = false
      }).catch(error => {
        console.log(error)
        self.error = error.message
        self.network = false
        if (error.response.data.code === 104) {
          self.$emit('logout')
        }
      })
    },
    defaultTable (data) {
      let header = [
        {
          text: 'Data',
          align: 'left',
          sortable: false,
          value: 'Data'
        },
        {
          text: 'Value',
          align: 'left',
          sortable: false,
          value: 'Value'
        }
      ]
      let name = ['Avg', 'Q1', 'Median', 'Q3', 'Max', 'Min']
      let table = []
      for (let i of name) {
        table.push({
          Data: i,
          Value: data[i]
        })
      }

      return ({header: header, data: table})
    },
    defaultOption (datas, categories) {
      return {
        chart: {
          type: 'spline'
        },
        title: {
          text: 'Wins Distribution (' + this.title + ')'
        },
        xAxis: {
          categories: categories,
          tickmarkPlacement: 'on'
        },
        yAxis: {
          title: {
            text: 'times'
          },
          type: 'logarithmic'
        },
        legend: {
          enabled: false
        },
        credits: {
          enabled: false
        },
        tooltip: {
          pointFormat: '{series.name}: <b>{point.y}</b>'
        },
        plotOptions: {
          spline: {
            marker: {
              enabled: false
            }
          }
        },
        series: [{
          name: 'Times',
          data: datas
        }]
      }
    },
    conertData (data) {
      let result = []
      this.categories = []
      for (let index in data) {
        if (data[index] !== 0) {
          result.push([index, data[index]])
          this.categories.push(parseFloat(index))
        }
      }
      this.categories.sort((x, y) => {
        if (x > y) return 1
        else if (x < y) return -1
        else return 0
      })
      result.sort(function (x, y) {
        if (parseFloat(x[0]) > parseFloat(y[0])) return 1
        else if (parseFloat(x[0]) < parseFloat(y[0])) return -1
        else return 0
      })
      return result
    },
    change: _.debounce(function () {
      this.start()
    }, 1000)
  }
}
</script>

<style scoped>
.inline>* {
  display: inline-block;
  vertical-align: top;
}
</style>