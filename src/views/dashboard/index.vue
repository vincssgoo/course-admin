<template>
  <div class="dashboard-container">
    <!-- <component :is="currentRole"/> -->
  </div>
</template>

<script>

import emptyDashboard from './empty'

export default {
  name: 'Dashboard',
  components: { emptyDashboard },
  data () {
    return {
      currentRole: 'emptyDashboard'
    }
  },
  computed: {

  },
  created () {
  }
}
</script>
