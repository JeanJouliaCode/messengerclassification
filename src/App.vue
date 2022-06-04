<template>
  <div id="app">
    <h1>Guess who</h1>
    <span>{{error}}</span>
    <textarea  type="text" @input="handleInput" cols="30" rows="5"/>
    <h2>{{label}}</h2>
    <div class="histo">
      <div ref="bar0" class="bar"><h3 class="name">{{labels[0]}}</h3></div>
      <div ref="bar1" class="bar"><h3 class="name">{{labels[1]}}</h3></div>
      <div ref="bar2" class="bar"><h3 class="name">{{labels[2]}}</h3></div>
      <div ref="bar3" class="bar"><h3 class="name">{{labels[3]}}</h3></div>
    </div>
  </div>
</template>

<script>

import * as tf from '@tensorflow/tfjs'
import words from './assets/words.json'

export default {
  name: 'App',
  data () {
    return {
      label: '‎',
      model: null,
      value: '',
      error: '‎',
      labels: ['PA', 'Benoît', 'Jean', 'Quentin']
    }
  },
  mounted () {
    this.createModel()
  },
  methods: {
    async createModel () {
      this.model = await tf.loadLayersModel('https://tensorflowmodeljean.s3.eu-west-1.amazonaws.com/model.json')
    },
    handleInput (val) {
      const formatedSentence = this.formatSentence(val.target.value)

      if (!formatedSentence) {
        this.label = '‎'
        this.error = '‎'
        this.setBar([5, 5, 5, 5])
        return
      }

      const result = this.model.predict(tf.tensor([formatedSentence]))

      let formatedResult = [result.dataSync()[0], result.dataSync()[1], result.dataSync()[2], result.dataSync()[3]]
      const maxValue = Math.max(...formatedResult)
      const minValue = Math.min(...formatedResult)

      this.label = this.labels[formatedResult.indexOf(maxValue)]

      formatedResult = formatedResult.map(r => (((r - minValue) / (maxValue - minValue)) * 100))
      this.setBar(formatedResult)
    },
    setBar (heights) {
      heights.forEach((height, index) => {
        this.$refs[`bar${index}`].style.height = height + '%'
      })
    },
    verifyValidity (list) {
      switch (true) {
        case list.length < 5:
          this.error = 'Please enter more than 5 allowed words for better result'
          break
        case list.length > 30:
          this.error = 'Please enter less than 30 words, words above 30 wont be considered'
          break
        default:
          this.error = '‎'
      }
    },
    formatSentence (sentence) {
      let newSentence = sentence.toLowerCase()
      const replaceObject = [['\\n', ' '], ['\n', ' '], ['.', ' '], ["'", ' '], [',', ' '], ['"', ''], [':', ' '], ['(', ' '], [')', ' '], ['!', ' !'], ['@', ' '], ['?', ' ?']]

      // replace unwanted character
      replaceObject.forEach(replace => {
        newSentence.replace(replace[0], replace[1])
      })

      // split sentence into array of word
      newSentence = newSentence.split(' ')
      newSentence = newSentence.filter(w => w !== '')

      // replace word by number
      newSentence = newSentence.map(word => {
        return words.list.indexOf(word)
      })

      // replace unwanted character
      newSentence = newSentence.filter(w => w >= 0)

      this.verifyValidity(newSentence)

      if (newSentence.length === 0) return null

      if (newSentence.length > 30) {
        newSentence = newSentence.slice(0, 30)
      }

      // padd the word with zero to go above 30
      const lengthSave = newSentence.length
      for (let i = 0; i < (30 - lengthSave); i++) {
        newSentence.push(0)
      }

      return newSentence
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Balsamiq+Sans&family=Montserrat:wght@300;500;600;700;800;900&family=Poppins&display=swap');

body{
  padding: 0px;
  margin: 0px;
}

#app {
  font-family: 'Poppins', sans-serif;
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
}

.histo{
  margin-top: 20px;
  width: 100%;
  max-width: 500px;
  height: 200px;
  display: flex;
  justify-content: space-around;
  align-items: flex-end;
}

.bar{
  width: 20%;
  height: 5%;
  background: rgb(255, 179, 93);
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  align-items: center;
  border-top-left-radius: 8px;
  border-top-right-radius: 8px;
}

.name{
  margin-bottom: -30px;
}

span{
  color: rgb(8, 8, 8);
}
</style>
