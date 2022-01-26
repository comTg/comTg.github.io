<template>
  <div class="about-me bga-back-top">
    <article class="markdown-body" v-html="renderedMarkdown"/>
  </div>
</template>
<script>
  export default {
    data () {
      return {
        renderedMarkdown: ''
      }
    },
    methods: {
      getReadme () {
        this.$gitHubApi.getReadme(this).then(response => {
          if (response.data) {
            this.renderedMarkdown = this.$marked(response.data)
          }
        })
      }
    },
    mounted () {
      this.$nextTick(() => {
        document.title = 'about me'
        this.getReadme()
      })
    }
  }
</script>
<style lang="scss" scoped>
  .about-me {
    width: calc(100% - 260px); // 兼容 IE 10
    flex-grow: 1;
    padding: 30px 100px;
    overflow-y: auto;
  }
</style>