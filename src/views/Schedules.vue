<template>
  <v-container fluid>
    <v-row>
      <v-col sm="12">
        <ScheduleTable :key="refreshKey"></ScheduleTable>
        <Fab v-if="fabs" :fabs="fabs" v-on:fab_action="fabAction"></Fab>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  import ScheduleTable from "../components/ScheduleTable"
  import Fab from "../components/core/Fab"

  export default {
    name: "Schedules",
    components: { Fab, ScheduleTable },
    data: () => ({
      fabs: [
        {
          color: "pink",
          action: "refreshSchedules",
          icon: "refresh",
          tooltip: "Refresh schedules",
        },
      ],
      refreshKey: 0,
    }),
    methods: {
      fabAction(action) {
        this[action]()
      },
      refreshSchedules() {
        this.$toast("refreshing schedules")
        this.$http.post("/api/schedules/refresh/").then(() => {
          this.refreshKey += 1
          this.$toast("schedules refreshed")
        }).catch((error) => {
          this.$toast.error(error.response.data)
        })

      },
    },
  }
</script>

<style scoped>

</style>