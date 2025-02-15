<template>
  <div class="dashboard">
    <onboarding
      class="dashboard-onboarding"
      v-if="isOnboarding"
      :isEditingProfile="isEditingProfile"
      :userName="userName"
      :onboardingData="onboardingData"
      @onboard="endOnboarding"
      @cancelOnboarding="cancelOnboarding"
    />
    <div class="dashboard-mainView">
      <div class="dashboard-menus">
        <nav-bar
          class="dashboard-nav"
          data-cyId="navbar"
          :isDisplayingRequirementsMobile="requirementsIsDisplayedMobile"
          @openPlan="openPlan"
          @openTools="openTools"
          @openProfile="openProfile"
          @toggleRequirementsMobile="toggleRequirementsMobile"
        />
        <requirement-side-bar
          class="dashboard-reqs"
          data-cyId="reqsSidebar"
          v-if="loaded && !showToolsPage && !isProfileOpen"
          :isMobile="isTablet"
          :isDisplayingMobile="requirementsIsDisplayedMobile"
          :isMinimized="requirementsIsMinimized"
          @toggleMinimized="toggleMinimizeRequirements"
          :startTour="startTour"
          @showTourEndWindow="showTourEnd"
        />
        <bottom-bar
          v-if="!(isTablet && requirementsIsDisplayedMobile) && !showToolsPage && !isProfileOpen"
          :isNavbarWide="requirementsIsMinimized"
          :isExpanded="bottomBarIsExpanded"
          :maxBottomBarTabs="maxBottomBarTabs"
        />
      </div>
      <semester-view
        v-if="
          loaded && !(isTablet && requirementsIsDisplayedMobile) && !showToolsPage && !isProfileOpen
        "
        ref="semesterview"
        :compact="compactVal"
        :startTour="startTour"
        :isBottomBarExpanded="bottomBarIsExpanded"
        :isBottomBar="hasBottomCourses"
        :isMobile="isMobile"
        @compact-updated="compactUpdated"
      />
      <tools-container class="toolsPage" v-if="showToolsPage" />
      <profile-editor
        class="profilePage"
        :onboardingData="onboardingData"
        :userName="userName"
        v-if="isProfileOpen"
      />
    </div>
    <tour-window
      title="Welcome to CoursePlan!"
      text="View your college requirements, plan your semesters and courses, and more."
      exit="No, I want to skip this"
      button-text="Start Tutorial"
      @startTour="startWelcomeTour"
      @closeTourWindow="closeWelcome"
      v-if="welcomeHidden"
    />
    <tour-window
      title="Let's get CoursePlanning!"
      text="There's more to explore as you start planning! CoursePlan is continously improving, so please use it as a guide and
      also consult your advisors for more up to date information!"
      :isFinalStep="true"
      exit=""
      button-text="Get Started"
      @closeTourWindow="closeTour"
      v-if="showTourEndWindow"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

import introJs from 'intro.js';
import SemesterView from '@/components/Semester/SemesterView.vue';
import RequirementSideBar from '@/components/Requirements/RequirementSideBar.vue';
import BottomBar from '@/components/BottomBar/BottomBar.vue';
import NavBar from '@/components/NavBar.vue';
import Onboarding from '@/components/Modals/Onboarding/Onboarding.vue';
import TourWindow from '@/components/Modals/TourWindow.vue';
import ToolsContainer from '@/containers/Tools.vue';
import ProfileEditor from '@/containers/Profile.vue';
import featureFlagCheckers from '@/feature-flags';

import store, { initializeFirestoreListeners } from '@/store';
import { immutableBottomBarState } from '@/components/BottomBar/BottomBarState';
import {
  smallBreakpoint,
  mediumBreakpoint,
  veryLargeBreakpoint,
} from '@/assets/constants/scss-variables';

const smallBreakpointPixels = parseInt(
  smallBreakpoint.substring(0, smallBreakpoint.length - 2),
  10
);
const mediumBreakpointPixels = parseInt(
  mediumBreakpoint.substring(0, mediumBreakpoint.length - 2),
  10
);
const veryLargeBreakpointPixels = parseInt(
  veryLargeBreakpoint.substring(0, veryLargeBreakpoint.length - 2),
  10
);

const getMaxButtonBarTabs = () => {
  if (window.innerWidth <= veryLargeBreakpointPixels) {
    return window.innerWidth <= smallBreakpointPixels ? 1 : 2;
  }
  return 4;
};

const tour = introJs();
tour.setOption('exitOnEsc', 'false');
tour.setOption('doneLabel', 'Next');
tour.setOption('nextLabel', 'Next');
tour.setOption('exitOnOverlayClick', 'false');

// eslint-disable-next-line @typescript-eslint/no-empty-function
let listenerUnsubscriber = (): void => {};

export default defineComponent({
  components: {
    BottomBar,
    NavBar,
    Onboarding,
    RequirementSideBar,
    SemesterView,
    TourWindow,
    ToolsContainer,
    ProfileEditor,
  },
  data() {
    return {
      loaded: true,
      compactVal: false,
      showSideBar: true,
      isOnboarding: false,
      isEditingProfile: false,
      isTablet: window.innerWidth <= mediumBreakpointPixels,
      isMobile: window.innerWidth <= smallBreakpointPixels,
      requirementsIsDisplayedMobile: false,
      requirementsIsMinimized: false,
      maxBottomBarTabs: getMaxButtonBarTabs(),
      welcomeHidden: false,
      startTour: false,
      showTourEndWindow: false,
      showToolsPage: false,
      isProfileOpen: false,
    };
  },
  computed: {
    userName(): FirestoreUserName {
      return store.state.userName;
    },
    onboardingData(): AppOnboardingData {
      return store.state.onboardingData;
    },
    semesters(): readonly FirestoreSemester[] {
      return store.state.semesters;
    },
    hasBottomCourses(): boolean {
      return immutableBottomBarState.bottomCourses.length > 0;
    },
    bottomBarIsExpanded(): boolean {
      return immutableBottomBarState.isExpanded;
    },
  },
  created() {
    window.addEventListener('resize', this.resizeEventHandler);
  },
  mounted() {
    listenerUnsubscriber = initializeFirestoreListeners(() => {
      if (this.onboardingData.college !== '' || this.onboardingData.grad !== '') {
        this.loaded = true;
      } else {
        this.startOnboarding();
      }
    });
  },
  unmounted() {
    window.removeEventListener('resize', this.resizeEventHandler);
    listenerUnsubscriber();
  },
  methods: {
    resizeEventHandler() {
      this.isMobile = window.innerWidth <= smallBreakpointPixels;
      this.isTablet = window.innerWidth <= mediumBreakpointPixels;
      this.maxBottomBarTabs = getMaxButtonBarTabs();
    },
    toggleRequirementsMobile() {
      this.requirementsIsDisplayedMobile = !this.requirementsIsDisplayedMobile;
    },
    toggleMinimizeRequirements() {
      this.requirementsIsMinimized = !this.requirementsIsMinimized;
    },
    compactUpdated(compact: boolean) {
      this.compactVal = compact;
    },

    showTourEnd() {
      if (!this.isMobile) {
        this.showTourEndWindow = true;
      }
    },

    startOnboarding() {
      this.isOnboarding = true;
      store.commit('setIsTeleportModalOpen', true);
    },

    endOnboarding() {
      if (!this.isMobile && !this.isEditingProfile) {
        this.welcomeHidden = true;
      }
      this.loaded = true;
      this.cancelOnboarding();
    },

    cancelOnboarding() {
      this.isOnboarding = false;
      store.commit('setIsTeleportModalOpen', false);
    },

    startWelcomeTour() {
      if (!this.startTour) {
        this.startTour = true;
        this.welcomeHidden = false;
      }
    },

    openPlan() {
      this.showToolsPage = false;
      this.isProfileOpen = false;
    },

    openTools() {
      this.showToolsPage = true;
      this.isProfileOpen = false;
    },

    editProfile() {
      this.isEditingProfile = true;
      this.startOnboarding();
    },

    openProfile() {
      if (featureFlagCheckers.isProfileEnabled()) {
        this.isProfileOpen = true;
        this.showToolsPage = false;
      } else {
        this.editProfile();
      }
    },

    closeWelcome() {
      this.welcomeHidden = false;
    },

    closeTour() {
      this.showTourEndWindow = false;
    },
  },
});
</script>

<style scoped lang="scss">
@import '@/assets/scss/_variables.scss';

.dashboard {
  display: flex;
  flex-direction: column;

  &-mainView {
    display: flex;
    background-color: $backgroundBlue;
    min-height: 100vh;
  }

  &-menus {
    display: flex;
  }

  /* The Modal (background) */
  &-onboarding {
    position: fixed; /* Stay in place */
    z-index: 4; /* Sit on top */
    left: 0;
    top: 0;
    width: 100%; /* Full width */
    height: 100%; /* Full height */
    overflow: auto; /* Enable scroll if needed */
    background-color: rgb(0, 0, 0); /* Fallback color */
    background-color: rgba(0, 0, 0, 0.4); /* Black w/ opacity */
  }
  .emoji-text {
    height: 14px;
  }
}

.toolsPage {
  width: 100%;
}

.profilePage {
  width: 100%;
}

.semester {
  margin: 1rem;
  padding: 1rem;
  height: 12.12rem;
}

@media only screen and (max-width: $medium-breakpoint) {
  .dashboard {
    &-reqs {
      margin-left: 0;
    }
  }
}
</style>
