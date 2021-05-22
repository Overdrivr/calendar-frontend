<template>
  <div class="hello">
    <b-modal
      ref="modal-confirm-draft"
      title="Create a meeting"
      @cancel="draftMeetingModalCancel"
      @ok="confirmDraftMeeting">
      <h2 class="my-4">Your Meeting</h2>
        {{displayedMeetingDraftDate}}
    </b-modal>

    <b-modal
      ref="modal-display-meeting"
      title="Your meeting">
      <h2 class="my-4">Your Meeting</h2>
      <p>{{displayedMeetingDraftDate}}</p>
      <p>Zoom call : <a target="_blank" :href="displayedMeetingDraft.url">{{displayedMeetingDraft.url}}</a></p>
    </b-modal>

    <div class="headers d-flex justify-content-between  mx-4">
      <div class="day-header" v-for="day in weekdays">
        <h3>{{day}}</h3>
      </div>
    </div>

    <div class="cells d-flex justify-content-between mx-4">
      <div class="slots-column p-2" v-for="day in weekdays">
        <div
          class="slot mt-2 p-2 drop-area" v-for="slot in hourlySlots"
          @dragover.prevent @dragenter.prevent
          @drop="endDrag(day, slot)"
          v-on:click="createMeetingDraft(day, slot)">
          {{slot}}:00 > {{slot+1}}:00
          <div class="meeting-slot" v-if="hasMeeting(day, slot)" draggable
            @drag="startDrag(day, slot)">
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
      displayedMeetingDraft: {},
      draggedItem: {}
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
    findMeeting(day, slot) {
      for (let i = 0 ; i < this.meetings.length ; i++) {
        let meeting = this.meetings[i]
        if (meeting.day === day && meeting.slot === slot) {
          return meeting
        }
      }
    },
    hasMeeting(day, slot) {
      let meeting = this.findMeeting(day, slot)
      return !!(meeting)
    },
    draftMeetingModalCancel() {
      console.log('Modal closed')
      this.removeMeeting(this.displayedMeetingDraft)
    },
    confirmDraftMeeting() {
      console.log(`Confirming draft meeting : ${this.displayedMeetingDraft}`)
      console.log(`Meeting planned for : ${this.displayedMeetingDraftDate}`)

      let startDate = this.daySlotToDate(
        this.displayedMeetingDraft.day,
        this.displayedMeetingDraft.slot
      )

      console.log(startDate)

      this.displayedMeetingDraft.status = 'confirmed'

      axios
        .post('http://localhost:5000', {
          'startDate': startDate.toISOString()
        })
        .then(res => {
          console.log('Confirmed meeting :', res)
          this.displayedMeetingDraft.url = res.data.join_url
          this.$refs['modal-display-meeting'].show()
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
      let meetingStart = new Date(2021, 6, 5 + dayOffset, slot, 0, 0)
      //let meetingEnd = Date(2021, 7, 5 + dayOffset, slot + 1, 0 ,0)

      return meetingStart
    },
    startDrag(day, slot) {
      console.log('Starting drag', day, slot)
      this.draggedItem = this.findMeeting(day, slot)
    },
    endDrag (day, slot) {
      console.log('Ending drag ', this.draggedItem, 'to', day, slot)
      this.draggedItem.day = day
      this.draggedItem.slot = slot

      // A bit of a hack to simplify code and force regen when meeting is dragged
      this.displayedMeetingDraft = this.draggedItem
      this.confirmDraftMeeting()

      this.draggedItem = {}
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
