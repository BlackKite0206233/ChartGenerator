<template>
  <v-layout justify-center>
    <v-flex xs10 v-if="uploadError">
      error
    </v-flex>
    <v-flex xs10 v-else-if="isSuccess">
      <v-card>
        <v-layout row wrap pa-4 ma-4>
            <v-flex xs1></v-flex>
            <v-flex xs6 pb-4>
              <v-flex xs6>
                <file-input label="BaseGame Simulation Par Sheet" accept=".csv" @file="BaseSimPar"></file-input>
                <file-input label="BonusGame Simulation Par Sheet" accept=".csv" @file="BonusSimPar"></file-input>
                <file-input label="OverAll Simulation Par Sheet" accept=".csv" @file="OverallSimPar"></file-input>
                <file-input label="BaseGame Spin Data" accept=".csv" @file="BaseSpinData"></file-input>
                <file-input label="BonusGame Spin Data" accept=".csv" @file="BonusSpinData"></file-input>
                <file-input label="OverAll Spin Data" accept=".csv" @file="OverallSpinData"></file-input>
              </v-flex>
            </v-flex>
            <v-flex xs5 pb-4>
              <v-flex xs7>
                <file-input label="BaseGame Theory Par Sheet" accept=".csv" @file="BaseTheoryPar"></file-input>
                <file-input label="BonusGame Theory Par Sheet" accept=".csv" @file="BonusTheoryPar"></file-input>
                <file-input label="OverAll Theory Par Sheet" accept=".csv" @file="OverallTheoryPar"></file-input>
                <file-input label="OverAll Survival Rate" accept=".csv" @file="OverallSurvivalRate"></file-input>
              </v-flex>
            </v-flex>
            <v-flex xs1></v-flex>
            <v-btn class="blue darken-2 grey--text text--lighten-4" @click="send">SEND</v-btn>
        </v-layout>
      </v-card>
    </v-flex>
    <v-progress-circular indeterminate class="primary--text" v-else :size="50" style="width:100%;"></v-progress-circular>
  </v-layout>
</template>

<script>
import api from '../../store/api'
export default {
  data () {
    return {
      files: ['baseSimPar', 'bonusSimPar', 'overallSimPar', 'baseSpinData', 'bonusSpinData', 'overallSpinData', 'overallSurvivalRate', 'baseTheoryPar', 'bonusTheoryPar', 'overallTheoryPar'],
      baseSimPar: null,
      bonusSimPar: null,
      overallSimPar: null,
      baseTheoryPar: null,
      bonusTheoryPar: null,
      overallTheoryPar: null,
      baseSpinData: null,
      bonusSpinData: null,
      overallSpinData: null,
      overallSurvivalRate: null,
      isSuccess: true,
      uploadError: false
    }
  },
  methods: {
    send () {
      this.isSuccess   = false
      this.uploadError = false
      let promises = []
      for (let file of this.files) {
        if (this[file] !== null) {
          let form = new FormData()
          form.append(file, this[file], this[file].name)
          form.append('name', file)
          promises.push(api.uploadFiles(localStorage.getItem('token'), this.$store.state.projectId.id, form))
        }
      }
      Promise.all(promises).then(() => {
        this.isSuccess = true
        this.$router.push('/Project/')
        console.log('success')
      }).catch(error => {
        this.uploadError = true
        console.log(error.message)
      })
    },
    BaseSimPar (data) {
      this.baseSimPar = data
    },
    BonusSimPar (data) {
      this.bonusSimPar = data
    },
    OverallSimPar (data) {
      this.overallSimPar = data
    },
    BaseTheoryPar (data) {
      this.baseTheoryPar = data
    },
    BonusTheoryPar (data) {
      this.bonusTheoryPar = data
    },
    OverallTheoryPar (data) {
      this.overallTheoryPar = data
    },
    BaseSpinData (data) {
      this.baseSpinData = data
    },
    BonusSpinData (data) {
      this.bonusSpinData = data
    },
    OverallSpinData (data) {
      this.overallSpinData = data
    },
    OverallSurvivalRate (data) {
      this.overallSurvivalRate = data
    }
  }
}
</script>
