<template>
  <div id="app">
    <div class="ui inverted dimmer" :class="{'active': showModal}">
      <div class="ui modal" :class="{'active': showModal}">
        <i class="close icon"></i>
        <div class="header">
          Add Alarm
        </div>
        <div class="content">
          <form class="ui form">
            <div class="field">
              <label>Name</label>
              <input type="text" v-model="newAlarm.name" placeholder="Name for Alarm">
            </div>
                
            <div class="field">
              <label>Link to Audio File (must be available on web)</label>
              <input type="text" v-model="newAlarm.audio" placeholder="Audio File">
            </div>
            <div class="four fields">
              <div class="field">
                <label>Hour</label>
                <select class="ui fluid dropdown" v-model="newAlarm.time.hour">
                  <option :value="hour" v-for="hour in hours">{{hour}}</option>
                </select>
              </div>
              <div class="field">
                <label>Minute</label>
                <select class="ui fluid dropdown" v-model="newAlarm.time.minute">
                  <option :value="minute" v-for="minute in minutes">{{minute}}</option>
                </select>
              </div>
              <div class="field">
                <label>Day Part</label>
                <select class="ui fluid dropdown" v-model="newAlarm.time.dayPart">
                  <option value="am">am</option>
                  <option value="pm">pm</option>
                </select>
              </div>
              <div class="field">
                <label>Day of Week</label>
                <select class="ui fluid dropdown" v-model="newAlarm.day">
                  <option value="Monday">Monday</option>
                  <option value="Tuesday">Tuesday</option>
                  <option value="Wednesday">Wednesday</option>
                  <option value="Thursday">Thursday</option>
                  <option value="Friday">Friday</option>
                  <option value="Saturday">Saturday</option>
                  <option value="Sunday">Sunday</option>
                </select>
              </div>
            </div>
          </form>
        </div>
        <div class="actions">
          <div class="ui button" @click="showModal = false">Cancel</div>
          <div class="ui button green" @click="addAlarm()">Add</div>
        </div>
      </div>
    </div>
    
    <h2 class="ui icon header" style="width:100%; margin-top:50px;">
      <i class="clock icon"></i>
      <div class="content">
        School bell for curtis
      </div>
    </h2>
    <h3 class="ui icon header" style="width:100%;">
      <div class="ui statistic">
        <div class="label">
          {{ currentDate }}
        </div>
        <div class="value">
          {{ currentTime }}
        </div>
      </div>
    </h3>
    <div class="ui raised segment">
      <table class="ui compact celled definition table" v-if="alarms.length > 0">
        <thead>
          <tr>
            <th></th>
            <th>Name</th>
            <th>Day</th>
            <th>Time</th>
            <th>Audio</th>
            <th>Delete</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(alarm, index) in alarms">
            <td class="collapsing">
              <div class="ui fitted slider checkbox">
                <input type="checkbox" v-model="alarm.active"> <label></label>
              </div>
            </td>
            <td>{{alarm.name}}</td>
            <td>{{alarm.day}}</td>
            <td>{{alarm.time.hour}}:{{alarm.time.minute}} {{alarm.time.dayPart}}</td>
            <td>
              <button class="ui icon button secondary" @click="testAlarm(index)">
                <i class="bell outline icon"></i>
              </button>
              Push to Test
            </td>
            <td>
              <button class="ui icon button red" @click="deleteAlarm(index)">
                <i class="trash icon"></i>
              </button>
            </td>
          </tr>
        </tbody>
        <tfoot class="full-width">
          <tr>
            <th></th>
            <th colspan="5">
              <div class="ui right floated small primary labeled icon button" @click="showModal = !showModal">
                <i class="clock outline icon"></i> Add Alarm
              </div>
            </th>
          </tr>
        </tfoot>
      </table>
      <div class="ui info message" v-else>
        <div class="header">
          Add an alarm
        </div>
        <p>
          Go ahead. Do it.
        </p>
        <div class="ui primary labeled icon button" @click="showModal = !showModal">
          <i class="clock outline icon"></i> Add Alarm
        </div>
      </div>
    </div>
    
  </div>
</template>

<script>
import moment from 'moment'
export default {
  name: 'app',
  data () {
    return {
      showModal: false,
      clockIsOn: false,
      currentTime: '',
      currentDate: '',
      digestCycleId: false,
      newAlarm: {
        active: false,
        name: 'Test',
        day: 'Friday',
        time: {
          hour: 2,
          minute: 20,
          dayPart: 'am'
        },
        audio: 'https://s3.us-east-2.amazonaws.com/myground/184149__wancle__ac-bel.wav'
      },
      alarms: JSON.parse(localStorage.getItem('alarms')) || []
    }
  },
  methods: {
    accounceTime () {
      this.currentTime = moment().format('h:mm:ss a')
      this.currentDate = moment().format('ddd MMM Do')
    },
    testAlarm (index) {
      var vm = this;
      console.log('testing');
      
      let test = new Audio(vm.alarms[index].audio)
      test.play()
    },
    deleteAlarm (index) {
      let local = JSON.parse(localStorage.getItem('alarms'))
      local.splice(index, 1)
      localStorage.setItem('alarms', JSON.stringify(local))
      this.alarms = local
    },
    addAlarm () {
      console.log(localStorage);
      
      let local = JSON.parse(localStorage.getItem('alarms'))
      local.push(this.newAlarm)
      localStorage.setItem('alarms', JSON.stringify(local))
      this.alarms = local
      this.showModal = false
    }
  },
  computed: {
    hours () {
      const hours = []
      for (let i = 0; i < 12; i++) {
          hours.push(i + 1)
      }
      return hours
    },
    minutes () {
      const minutes = []
      for (let i = 0; i < 60; i++) {
        if (i < 9) {
          minutes.push('0' + (i))
        } else {
          minutes.push(i)
        }
      }
      return minutes
    }
  },
  mounted () {
    var vm = this
    if (!localStorage.getItem('alarms')) {
      localStorage.setItem('alarms', '[]')
    }
    vm.accounceTime()
    vm.digestCycleId = setInterval(function () {
      vm.accounceTime()
      vm.alarms.map(function (each) {
        let today = moment()
        let weekday = today.format('dddd')
        let hour = today.format('h')
        let minute = today.format('mm')
        let second = today.format('ss')
        let dayPart = today.format('a')
        // console.log(weekday, hour, minute, second, dayPart);
        // console.log(each.day, each.time.hour, each.time.minute, '00', each.time.dayPart);
        // console.log(each.active);
        if (each.active && each.day === weekday && each.time.hour == hour && each.time.minute == minute && '00' == second && each.time.dayPart === dayPart) {
          let audio = new Audio(each.audio)
          audio.play()
        }
      })
    }, 1000)
  }
}
</script>

<style lang="css" scoped>

</style> 