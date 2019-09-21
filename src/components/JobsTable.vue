<template>
  <v-container>
    <v-row no-gutters v-if="filter == null">
      <v-col sm="12">
        <v-card class="mb-8">
          <v-row>
            <v-col lg="2">
              <v-card-title>Search Jobs</v-card-title>
            </v-col>
            <v-divider></v-divider>
            <v-col lg="2" align-self="center">
              <vc-date-picker
                  :max-date='new Date()'
                  v-model="selectedDate"
                  mode="range"
                  :input-props='{ class: "input", placeholder: "Select Date(s)"}'
                  :formats="{input: ['YYYY-MM-DD']}"
              ></vc-date-picker>
            </v-col>
            <v-col lg="2">
              <v-select
                  :items="users"
                  v-model="selectedUsers"
                  placeholder="User(s)"
                  multiple
                  single-line
              >
              </v-select>
            </v-col>
            <v-col lg="2">
              <v-autocomplete
                  :items="minions"
                  v-model="selectedTarget"
                  placeholder="Target(s)"
                  multiple
                  single-line
              >
              </v-autocomplete>
            </v-col>
            <v-col lg="1">
              <v-select
                  :items="limit"
                  v-model="selectedLimit"
                  placeholder="Limit"
                  single-line
              >
              </v-select>
            </v-col>
            <v-col lg="1" align-self="center">
              <div class="text-center">
                <v-btn
                    color="primary"
                    @click="filterJobs"
                >Search
                </v-btn>
              </div>
            </v-col>
          </v-row>
        </v-card>
      </v-col>
    </v-row>
    <v-row no-gutters>
      <v-col sm="12">
        <v-card :elevation="filter == null ? 2 : 0">
          <v-card-title>
            Jobs
            <v-spacer></v-spacer>
            <v-text-field
                v-model="search"
                append-icon="search"
                label="Search"
                single-line
                hide-details
            ></v-text-field>
          </v-card-title>
          <v-data-table
              sort-by="jid"
              sort-desc
              item-key="uniqueid"
              :headers="headers"
              :items="indexedItems"
              :search="search"
              class="elevation-1"
              :loading="loading"
          >
            <template v-slot:item.jid="{ item }">
              <v-btn text small class="text-none" :to="'/jobs/'+item.jid+'/'+item.id">{{ item.jid }}</v-btn>
            </template>
            <template v-slot:item.arguments="{ item }">
              {{ item.arguments.length > 20 ? item.arguments.slice(0, 20)+'...': item.arguments }}
            </template>
            <template v-slot:item.success="{ item }">
              <v-chip :color="boolRepr(item.success)" dark>{{ boolText(item.success) }}</v-chip>
            </template>
            <template v-slot:item.alter_time="{ item }">
              {{new Date(item.alter_time).toLocaleString("en-GB")}}
            </template>
            <template v-slot:item.action="{ item }">
              <div class="text-center">
                <v-btn
                    small
                    class="ma-2"
                    color="blue"
                    tile
                    dark
                    :to="'/jobs/'+item.jid+'/'+item.id"
                >
                  detail
                </v-btn>
                <v-btn
                    small
                    class="ma-2"
                    color="blue-grey"
                    tile
                    dark
                    :to="'/run/?target='+item.id+'&function='+item.fun+'&args='+item.arguments"
                >
                  rerun
                </v-btn>
              </div>
            </template>
          </v-data-table>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>

  export default {
    name: "JobsTable",
    props: ["filter"],
    data() {
      return {
        limit: [50, 100, 200, 500, 1000],
        selectedDate: [new Date(), new Date()],
        selectedLimit: null,
        selectedUsers: null,
        selectedTarget: null,
        minions: [],
        users: [],
        search: "",
        headers: [
          { text: "Jid", value: "jid" },
          { text: "Target", value: "id" },
          { text: "Function", value: "fun" },
          { text: "Arguments", value: "arguments" },
          { text: "User", value: "user" },
          { text: "Status", value: "success" },
          { text: "Date", value: "alter_time" },
          { text: "Actions", value: "action", sortable: false },
        ],
        jobs: [],
        loading: true,
      }
    },
    computed: {
      indexedItems() {
        return this.jobs.map((item, index) => ({
          uniqueid: index,
          ...item,
        }))
      },
    },
    mounted() {
      this.loadData()
    },
    methods: {
      loadData() {
        this.$http.get("api/jobs/filters/").then(response => {
          this.minions = response.data.minions
          this.users = response.data.users
        })
        this.$http.get("api/jobs/", { params: this.filter }).then(response => {
          this.jobs = response.data
          this.loading = false
        })
      },
      filterJobs() {
        this.loading = true
        let params = {
          limit: this.selectedLimit,
          target: this.selectedTarget,
          users: this.selectedUsers,
        }
        if (this.selectedDate.start && this.selectedDate.end) {
          params.start = this.selectedDate.start.toISOString().slice(0, 10)
          params.end = this.selectedDate.end.toISOString().slice(0, 10)
        }
        this.$http.get("api/jobs/", {
          params: params,
        }).then(response => {
          this.jobs = response.data
          this.loading = false
          this.selectedUsers = this.selectedTarget = this.selectedLimit = this.selectedDate = null
          this.selectedDate = [new Date(), new Date()]
        })
      },
      boolRepr(bool) {
        if (bool === true) return "green"
        else return "red"
      },
      boolText(bool) {
        if (bool === true) return "success"
        else return "failed"
      },
    },
  }
</script>

<style scoped>

</style>