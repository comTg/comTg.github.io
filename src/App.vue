<template>
  <div class="app">
    <left-layout class="left-container"/>
    <router-view class="main-container"/>
  </div>
</template>
<script>
  import LeftLayout from './components/LeftLayout.vue'
  import { mapActions } from 'vuex'

  export default {
    components: {
      LeftLayout
    },
    methods: {
      ...mapActions([
        'setLabels',
        'setGitHubUser'
      ])
    },
    mounted () {
      this.$nextTick(function () {
        this.$gitHubApi.getUserInfo(this).then(this.$http.spread((userResp, labelResp) => {
          this.setGitHubUser(userResp.data)
          this.setLabels(labelResp.data)
        }))
      })
    }
  }
</script>
<style lang="scss" scoped>
  body {
    overflow: hidden;
  }
  .app {
    display: flex;
    height: 100vh;
    width: 100vw;
    overflow: hidden;
    font-smooth: auto;
    -webkit-font-smoothing: antialiased;
  }

  .left-container {
    border-right: 1px solid #eeeeee;
    height: 100%;
  }

  .main-container {
    flex-grow: 1;
  }
</style>