<template>
  <div class="toolsContainer">
    <div class="toolsContainer-header">
      <div class="toolsContainer-title">
        <span>Hi, {{ userInfo.firstName }}!</span>
      </div>
      <div class="toolsContainer-subtitle">
        <span>Here are some resources to help you plan.</span>
      </div>
    </div>
    <div class="toolsContainer-cards">
      <card name="Progress Tracker" class="toolsContainer-card-progress" id="progress">
        <progress-tracker />
      </card>
      <card name="Contact Your Advisors" class="toolsContainer-card-advisors" id="advisors">
        <advisor-card></advisor-card>
      </card>

      <card name="Export Schedule" class="toolsContainer-card-export" id="export">
        <export-card></export-card>
      </card>

      <card name="Useful Links" class="toolsContainer-card-links" id="links">
        <useful-links></useful-links>
      </card>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import Card from '@/components/Tools/Card.vue';
import AdvisorCard from '@/components/Tools/AdvisorCard.vue';
import UsefulLinks from '@/components/Tools/UsefulLinks.vue';
import ProgressTracker from '@/components/Tools/ProgressTracker.vue';
import ExportCard from '@/components/Tools/ExportCard.vue';
import store from '@/store';

export default defineComponent({
  components: { ProgressTracker, ExportCard, UsefulLinks, AdvisorCard, Card },
  computed: {
    userInfo(): FirestoreUserName {
      return store.state.userName;
    },
  },
});
</script>

<style scoped lang="scss">
@import '@/assets/scss/_variables.scss';

.toolsContainer {
  @media only screen and (max-width: $medium-breakpoint) {
    padding: 5rem 4rem;
  }
  background-color: #f6fafc;
  padding: 3rem 7rem;

  &-title {
    min-width: 100%;
    font-size: 36px;
    font-weight: bold;
    color: #3c3c3c;
    padding-bottom: 21px;
  }

  &-subtitle {
    font-size: 24px;
    color: #757575;
    padding-bottom: 30px;
    overflow: hidden;
  }

  &-cards {
    display: grid;
    grid-auto-columns: min-content;

    column-gap: 5rem;
    row-gap: 3rem;

    @media only screen and (max-width: 1250px) {
      display: flex;
      flex-wrap: wrap;
      justify-content: flex-start;
    }
  }

  &-card {
    &-progress {
      grid-column: 1/2;
      grid-row: 1/2;
      width: 600px;
      min-height: 250px;
    }

    &-advisors {
      grid-column: 1/2;
      grid-row: 2/3;
      width: 600px;
      height: min-content;
    }

    &-export {
      grid-column: 2/3;
      grid-row: 1/2;
      width: 300px;
      height: 250px;
    }

    &-links {
      grid-column: 2/3;
      grid-row: 2/3;
      width: 300px;
      height: 250px;
    }
  }
}
</style>
