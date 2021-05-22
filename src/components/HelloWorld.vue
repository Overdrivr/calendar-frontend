<template>
  <div class="hello">
    <b-modal
      ref="modal-confirm-draft"
      title="Meeting"
      @cancel="draftMeetingModalCancel"
      @ok="confirmDraftMeeting">
      <h2 class="my-4">Your Meeting</h2>

        {{displayedMeetingDraftDate}}
    </b-modal>

    <div class="headers d-flex justify-content-between  mx-4">
      <div class="day-header" v-for="day in weekdays">
        <h3>{{day}}</h3>
      </div>
    </div>

    <div class="cells d-flex justify-content-between mx-4">
      <div class="slots-column p-2" v-for="day in weekdays">
        <div
          class="slot mt-2 p-2" v-for="slot in hourlySlots"
          v-on:click="createMeetingDraft(day, slot)">
          {{slot}}:00 > {{slot+1}}:00
          <div class="meeting-slot" v-if="hasMeeting(day, slot)">
            MEETING
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
var axios = require('axios')

export default {
  name: 'HelloWorld',
  data () {
    return {
      weekdays: ['Mon', 'Tues', 'Wed', 'Thurs', 'Friday', 'Sat', 'Sun'],
      hourlySlots : this.generateDailySlots(),
      meetings: [],
      displayedMeetingDraft: {}
    }
  },
  methods: {
    createMeetingDraft(day, slot) {
      console.log(`Creating meeting draft for ${day} : ${slot}`)

      let newMeeting = {
        day,
        slot,
        status: 'draft',
        url: null
      }

      this.meetings.push(newMeeting)
      this.displayedMeetingDraft = newMeeting

      this.$refs['modal-confirm-draft'].show()
    },
    hasMeeting(day, slot) {
      for (let i = 0 ; i < this.meetings.length ; i++) {
        let meeting = this.meetings[i]
        if (meeting.day === day && meeting.slot === slot) {
          return true
        }
      }
      return false
    },
    draftMeetingModalCancel() {
      console.log('Modal closed')
      this.removeMeeting(this.displayedMeetingDraft)
    },
    confirmDraftMeeting() {
      console.log(`Confirming draft meeting : ${this.displayedMeetingDraft}`)
      console.log(`Meeting planned for : ${this.displayedMeetingDraftDate}`)

      this.displayedMeetingDraft.status = 'confirmed'

      axios
        .post('localhost:8000/', {
          'meetingDate': this.displayedMeetingDraftDate
        })
        .then(res => {
          console.log('Confirmed meeting :', res)
          this.displayedMeetingDraft.url = res.data.url
          return res
        })
        .catch(err => {
          console.warn('Request error: ', err)
        })
    },
    removeMeeting(meetingToRemove) {
      this.meetings = this.meetings.filter(el => el !== meetingToRemove)
    },

    generateDailySlots() {
      let slots = []

      for (let i = 5 ; i < 22 ; i++) {
        slots.push(i)
      }

      return slots
    },
    daySlotToDate(day, slot) {
      let dayOffset = this.weekdays.indexOf(day)

      // An arbitrary monday - July 5th
      let meetingStart = Date(2021, 7, 5 + dayOffset, slot, 0, 0)
      //let meetingEnd = Date(2021, 7, 5 + dayOffset, slot + 1, 0 ,0)

      return meetingStart
    }
  },
  computed : {
    displayedMeetingDraftDate () {
      return this.daySlotToDate(
        this.displayedMeetingDraft.day,
        this.displayedMeetingDraft.slot
      )
    }
  }
}
</script>

<style scoped>
.slots-column {
  width: 100%;
}
.slot {
  background-color: rgb(200, 200, 255, 0.3);
  width: 100%;
  min-height: 80px;
}
.meeting-slot {
  background-color: blue;
  width: 100%;
  height: 100%
}
</style>
