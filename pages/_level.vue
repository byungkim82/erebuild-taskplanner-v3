<template>
  <v-app>
    <v-system-bar app window dark color="primary">
      <v-icon color="white">mdi-file-tree-outline</v-icon>
      <span class="white--text font-weight-medium mr-2">
        {{ appTitle }}: {{ gameLevel }}
      </span>
      <v-spacer></v-spacer>
      <v-icon color="white">mdi-account-circle-outline</v-icon>
      <span class="white--text font-weight-medium"
        >{{ userToken }} | {{ competency }}</span
      >
    </v-system-bar>
    <v-navigation-drawer app permanent width="90">
      <v-list dense nav class="pa-0 mt-8">
        <v-list-item-group color="indigo" mandatory>
          <v-list-item
            v-for="(action, index) in tabs"
            :key="'action-tab-' + index"
            :nuxt="true"
            :to="action.link"
          >
            <v-list-item-content>
              <v-list-item-title style="font-size: 0.75rem">
                {{ action.title }}
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list-item-group>
      </v-list>
    </v-navigation-drawer>
    <v-main>
      <nuxt-child keep-alive></nuxt-child>
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: 'Level',
  data: () => ({
    appTitle: 'Task Planner',
    gameLevel: '',
    userToken: '',
    actions: [],
    taskPlannerEvents: [],
    opened: 0,
  }),
  computed: {
    tabs() {
      return this.actions.map((action) => ({
        title: action.toUpperCase(),
        link: `/${this.gameLevel}/${action}/`,
      }))
    },
    // userToken() {
    //   return this.$store.getters.userToken
    // },
    competency() {
      return this.$store.getters.competency
    },
    performance() {
      return this.$store.getters.performance
    },
  },
  beforeCreate() {
    const levelStr = this.$route.params.level
    if (levelStr) {
      this.$store.commit('setLevel', levelStr)
    }
    const userEmail = this.$route.query.email
    if (userEmail) {
      this.$store.commit('setUserToken', userEmail)
    }
  },
  created() {
    const levelStr = this.$route.params.level
    const userToken = this.$route.query.email
    const actions = this.$store.getters['data/planners'](levelStr)
    this.getPerformanceData(userToken, levelStr)
    // this.$store.commit('setLevel', levelStr)
    // this.$store.commit('setUserToken', userToken)
    this.gameLevel = levelStr
    this.userToken = userToken
    if (actions.length > 0) this.actions = actions
  },
  beforeMount() {
    window.addEventListener('message', this.sendPanelLogs)
  },
  destroyed() {
    window.removeEventListener('message', this.sendPanelLogs)
  },
  methods: {
    async getPerformanceData(userToken, gameLevel) {
      return await this.$axios
        .get(
          `https://mileresearch.coe.fsu.edu/adaptiveSupport/GetUserPerformance.php?email=${userToken}&level=${gameLevel}`
        )
        .then((res) => this.$store.commit('setPerformance', res.data))
    },
    async sendOpenLogs(userToken, gameLevel) {
      return await this.$axios.get(
        `https://mileresearch.coe.fsu.edu/taskplanner/analytics/panel/open/${gameLevel}/${userToken}`
      )
    },
    async sendCloseLogs(userToken, gameLevel) {
      return await this.$axios.get(
        `https://mileresearch.coe.fsu.edu/taskplanner/analytics/panel/close/${gameLevel}/${userToken}`
      )
    },
    async sendTaskPlannerRecords(data) {
      return await this.$axios.post(
        'https://mileresearch.coe.fsu.edu/taskplanner/analytics/panel/save/',
        data,
        {
          withCredentials: true,
        }
      )
    },
    sendPanelLogs(event) {
      // const userEmail = event.data.user_email
      const userEmail = this.$store.getters.userToken
      const levelName = this.$store.getters.level
      // this.$store.commit('setUserToken', userEmail)
      // this.getPerformanceData(userEmail, levelName)

      if (event.data.status === 'OPEN') {
        this.sendOpenLogs(userEmail, levelName)
        this.opened++
      } else if (event.data.status === 'CLOSE') {
        this.sendCloseLogs(userEmail, levelName)
        const data = {
          user_token: userEmail,
          game_level: levelName,
          data: JSON.stringify(this.$store.state.interactions),
        }
        this.sendTaskPlannerRecords(data)
        this.$store.commit('emptyInteractions')
      }
    },
  },
}
</script>

<style scoped></style>
