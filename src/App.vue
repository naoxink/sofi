<template>
  <div id="app">
    <div class="sentence-showing" v-if="sentenceShowing">
      <span>{{ sentenceShowing.text }}</span>
      <div class="author" v-if="sentenceShowing.author">{{ sentenceShowing.author }}</div>
    </div>
    <button @click="checkIn()" class="btn-check-in" :disabled="isIn">Entrada</button>
    <button @click="checkOut()" class="btn-check-out" :disabled="!isIn">Salida</button>
    <div class="lists">
      <ul class="checks-list" v-if="this.checks.length">
        <li class="listh">
          <span>Entrada</span>
          <span>Salida</span>
          <span>Tiempo total</span>
          <span>Eliminar</span>
        </li>
        <li v-for="(dates, index) of this.revChecks" :key="index" :class="{ 'running': !dates.out }" :id="`check-${dates.id}`">
          <span class="check-in">{{ formatTime(dates.in) }}</span>
          <span class="check-out" v-if="dates.out">{{ formatTime(dates.out) }}</span>
          <span v-else></span>
          <span class="total-time" v-if="dates.out">{{ formatTime(dates.total) }}</span>
          <span class="total-time" v-else>{{ timerStr }}</span>
          <span class="remove-row" @click="removeCheck(dates.id)">&times;</span>
        </li>
        <li class="listf">
          <span></span>
          <span>Tiempo total</span>
          <span>{{ sumTotalTime }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import frases from '@/assets/frases.js'

export default {
  name: 'App',
  data: function(){
    return {
      checks: [],
      timer: null,
      now: Date.now(),
      frases,
      sentenceShowing: ''
    }
  },
  computed: {
    sumTotalTime(){
      const secs = this.checks.reduce((acc, check) => {
        return acc += check.total ? check.total.getSeconds() : 0
      }, 0)
      return this.formatTime(new Date(0, 0, 0, 0, 0, secs))
    },
    revChecks(){
      return this.checks.slice().reverse()
    },
    isIn(){
      return this.checks[this.checks.length - 1] && this.checks[this.checks.length - 1].in && !this.checks[this.checks.length - 1].out
    },
    isOut(){
      return !this.isIn
    },
    timerStr(){
      const check = this.checks[this.checks.length - 1]
      if(!check || !check.in || !this.isIn){
        return false
      }
      const diff = Math.abs(this.now - check.in.getTime())
      return this.formatTime(new Date(0, 0, 0, 0, 0, 0, diff))
    }
  },
  methods: {
    totalTime(checkIn, checkOut){
      return new Date(0, 0, 0, 0, 0, 0, checkOut - checkIn)
    },
    formatTime(date){
      return [
        date.getHours() < 10 ? '0' + date.getHours() : date.getHours(),
        date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes(),
        date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds(),
      ].join(':')
    },
    checkIn(){
      this.startTimer()
      this.checks.push({
        'id': Date.now(),
        'in': new Date(),
        'out': null,
        'total': null
      })
      this.showSentence()
      this.saveInLocalStorage()
    },
    checkOut(){
      this.stopTimer()
      const check = this.checks[this.checks.length - 1]
      this.checks[this.checks.length - 1].out = new Date()
      this.checks[this.checks.length - 1].total = this.totalTime(check.in, check.out)
      this.showSentence()
      this.saveInLocalStorage()
    },
    tickTimer(){
      this.now = Date.now()
      const totalTime = this.totalTime(this.checks[this.checks.length - 1].in, new Date())
      const mins = totalTime.getMinutes()
      const secs = totalTime.getSeconds()
      if(mins === 30 && secs === 0){
        this.showSentence()
      }
      this.startTimer()
    },
    startTimer(){
      this.timer = setTimeout(this.tickTimer.bind(this), 1000)
      this.now = Date.now()
      this.timerStart = Date.now()
    },
    stopTimer(){
      clearTimeout(this.timer)
      this.timer = null
      this.showSentence()
    },
    showSentence(){
      const r = this.random(0, this.frases.length)
      const sentence = this.frases[r]
      this.sentenceShowing = sentence
    },
    random(min, max){
      return Math.floor(Math.random() * (max - min) + min)
    },
    getLocalStorageKey(){
      const today = new Date()
      return `sophi-${today.getFullYear()}-${today.getMonth()+1}-${today.getDate()}`
    },
    saveInLocalStorage(){
      const key = this.getLocalStorageKey()
      window.localStorage.setItem(key, JSON.stringify(this.checks))
    },
    loadFromLocalStorage(){
      const key = this.getLocalStorageKey()
      const checks = window.localStorage.getItem(key)
      if(!checks) return false
      this.checks = JSON.parse(checks).map(check => {
        return {
          in: new Date(check.in),
          out: new Date(check.out),
          total: new Date(check.total),
          id: +check.id
        }
      })
    },
    removeCheck(checkId){
      this.checks = this.checks.filter(check => check.id !== checkId)
      this.saveInLocalStorage()
    }
  },
  mounted(){
    this.loadFromLocalStorage()
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 20px;
}
.lists {
  margin: 10px;
}
ul {
  list-style: none;
  vertical-align: bottom;
  display: inline-block;
  width: 40%;
  margin: 0;
  padding: 0;
}
.lists > ul > li.listh {
  background-color: #666;
  color: white;
  font-weight: bold;
  -webkit-border-radius: 5px 5px 0 0;
  -moz-border-radius: 5px 5px 0 0;
  -ms-border-radius: 5px 5px 0 0;
  -o-border-radius: 5px 5px 0 0;
  border-radius: 5px 5px 0 0;
}
.lists > ul > li.listf {
  background-color: #EEE;
  color: #2c3e50;
  font-weight: bold;
}
.lists > ul > li:last-child:not(.listh) {
  -webkit-border-radius: 0 0 5px 5px;
  -moz-border-radius: 0 0 5px 5px;
  -ms-border-radius: 0 0 5px 5px;
  -o-border-radius: 0 0 5px 5px;
  border-radius: 0 0 5px 5px;
}
.lists > ul > li {
  margin: 2px 0;
  text-align: left;
}
.lists > ul > li:not(:nth-child(2)){
  border-top: 1px solid #EEE;
}
li.running {
  background-color: rgba(154, 205, 50, .5);
  color: #2c3e50;
  font-weight: bold;
}
.lists > ul > li > span {
  text-align: center;
  width: 25%;
  margin: 0;
  padding: 5px 0;
  display: inline-block;
}
.sentence-showing {
  font-size: 2em;
  /*background-color: #FFEBCD;
  color: #8B0000;
  border: 1px solid rgba(0, 0, 0, .1);*/
  padding: 10px;
  -webkit-border-radius: 5px;
  -moz-border-radius: 5px;
  -ms-border-radius: 5px;
  -o-border-radius: 5px;
  border-radius: 5px;
  margin-bottom: 30px;
  max-width: 50%;
  margin: 0 auto;
  font-weight: bold;
}
.sentence-showing .author {
  font-size: .6em;
  text-align: right;
  margin-right: 10px;
  font-weight: normal;
}
.btn-check-in, .btn-check-out {
  padding: 10px 15px;
  margin: 10px;
  width: 150px;
  cursor: pointer;
  border: 1px solid rgba(0, 0, 0, .1);
  font-size: 1em;
  -webkit-border-radius: 5px;
  -moz-border-radius: 5px;
  -ms-border-radius: 5px;
  -o-border-radius: 5px;
  border-radius: 5px;
  font-weight: bold;
}
.btn-check-in:disabled, .btn-check-out:disabled {
  cursor: not-allowed;
  opacity: .5;
  background-color: #EEE;
  color: #666;
}
.btn-check-in {
  background-color: yellowgreen;
  color: white;
  border-color: rgba(0, 0, 0, .1);
}
.btn-check-in:hover {
  background-color: #669933;
}
.btn-check-out {
  background-color: lightsalmon;
  color: white;
  border-color: rgba(0, 0, 0, .1);
}
.btn-check-out:hover {
  background-color: #FF6633;
}
.remove-row {
  font-size: 1.5em;
  line-height: 1em;
  vertical-align: middle;
  cursor: pointer;
  color: red;
  transition: all .2s;
}
.remove-row:hover {
  font-weight: bold;
}

@media (max-width: 800px) {
  .sentence-showing {
    max-width: 95%;
  }
  ul {
    width: 95%;
  }
}
</style>
