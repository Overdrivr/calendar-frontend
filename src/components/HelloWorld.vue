<template>
  <div class="hello">
    <b-modal
      ref="modal-confirm-draft"
      title="Meeting"
      @cancel="draftMeetingModalCancel"
      @ok="confirmDraftMeeting">
      <p class="my-4">Draft</p>
      {{displayedMeetingDraft.day}}
      {{displayedMeetingDraft.slot}}
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
          {{day}} {{slot}}
          <div class="meeting-slot" v-if="hasMeeting(day, slot)">
            MEETING
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      weekdays: ['Mon', 'Tues', 'Wed', 'Thurs', 'Friday', 'Sat', 'Sun'],
      hourlySlots : [1, 2, 3],
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
        status: 'draft'
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
      console.log('Confirming draft meeting')
    },
    removeMeeting(meetingToRemove) {
      this.meetings = this.meetings.filter(el => el !== meetingToRemove)
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
