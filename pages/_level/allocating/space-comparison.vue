<template>
  <v-row class="py-1" justify="center" dense>
    <div class="d-flex justify-center flex-column">
      <items-list-panel
        v-for="(occupant, index) in occupants"
        :key="'item-' + index"
        class="ma-1"
        v-bind="{
          reverse: false,
          subItems: occupant.members,
          color: colors[index].str,
          hexColor: colors[index].hex,
          ...occupant,
        }"
      ></items-list-panel>
    </div>
    <div class="d-flex justify-center flex-column mx-2">
      <question-prompt v-bind="feedQuestionPrompt()" @proceeded="completed()">
        <template #activator="slotProps">
          <v-btn
            color="error"
            elevation="10"
            fab
            dark
            v-bind="slotProps.attrs"
            v-on="slotProps.on"
          >
            <v-icon>{{
              !checked
                ? 'mdi-exclamation-thick'
                : ableToAllocate
                ? 'mdi-less-than-or-equal'
                : 'mdi-greater-than-or-equal'
            }}</v-icon>
          </v-btn>
        </template>
      </question-prompt>
    </div>
    <div class="d-flex justify-center flex-column">
      <items-list-panel
        v-for="(dwelling, index) in dwellings"
        :key="'dwelling-' + index"
        class="ma-1"
        v-bind="{
          reverse: true,
          color: 'primary lighten-4',
          hexColor: 'primary',
          ...dwelling,
        }"
      >
      </items-list-panel>
    </div>
  </v-row>
</template>

<script>
import pluralize from 'pluralize'
import ItemsListPanel from '~/components/ItemsListPanel'
import QuestionPrompt from '~/components/QuestionPrompt'
export default {
  name: 'SpaceComparison',
  components: { ItemsListPanel, QuestionPrompt },
  data() {
    return {}
  },
  async fetch() {},
  computed: {
    checked() {
      return this.$store.state.allocating.tabs[2].checked
    },
    colors() {
      return this.$store.state.allocating.colors
    },
    occupants() {
      return this.$store.getters['allocating/getOccupants']
    },
    dwellings() {
      return this.$store.getters['allocating/getDwellings']
    },
    totalSpaceNeeded() {
      return this.occupants
        .map((occupant) => occupant.properties.totalSpace.value)
        .reduce((sum, val) => sum + val, 0)
    },
    totalSpaceProvided() {
      return this.dwellings
        .map((dwelling) => dwelling.properties.totalSpace.value)
        .reduce((sum, val) => sum + val, 0)
    },
    ableToAllocate() {
      return this.totalSpaceNeeded <= this.totalSpaceProvided
    },
  },
  watch: {
    '$route.params.level': '$fetch',
  },
  created() {},
  methods: {
    feedQuestionPrompt() {
      return {
        title: 'Comparing Space Provided and Space Needed',
        question: `Do you think you can allocate all the ${pluralize(
          this.occupants[0].name.split(' ').pop()
        )} in the inventory to the ${pluralize(
          this.dwellings[0].name
        )} provided?`,
        inputType: 'radio',
        hasUnit: false,
        measuringUnit: '',
        choices: [
          { key: 'choice-1', label: 'Yes', value: 'yes' },
          { key: 'choice-2', label: 'No', value: 'no' },
        ],
        answer: this.totalSpaceProvided >= this.totalSpaceNeeded ? 'yes' : 'no',
        hintFeedback: `Please check out which of the two (the total space needed for all the ${pluralize(
          this.occupants[0].name.split(' ').pop()
        )} or the total space provided by the ${pluralize(
          this.dwellings[0].name
        )} is bigger.`,
      }
    },
    completed() {
      this.$store.commit('allocating/checkPhase', 2)
    },
  },
}
</script>

<style scoped></style>
