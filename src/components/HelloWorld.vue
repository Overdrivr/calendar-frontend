<template>
  <div class="hello">
    <div class="headers d-flex justify-content-between  mx-4">
      <div class="day-header" v-for="day in weekdays">
        <h3>{{day}}</h3>
      </div>
    </div>

    <div class="cells d-flex justify-content-between mx-4">
      <div class="slots-column p-2" v-for="day in weekdays">
        <div
          class="slot mt-2" v-for="slot in hourlySlots"
          @clicked="createMeetingDraft(day, slot)">
          {{day}} {{slot}}
          <div v-if="hasMeeting(day, slot)">
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
      meetings: []
    }
  },
  methods: {
    createMeetingDraft(day, slot) {
      console.log(`Creating meeting draft for ${day} : ${slot}`)
      this.meetings.push({
        day,
        slot,
        status: 'draft'
      })
      // TODO: Launch popup
    },
    hasMeeting(day, slot) {
      
      for (let i = 0 ; i < this.meetings.length ; i++) {

      }
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
</style>
