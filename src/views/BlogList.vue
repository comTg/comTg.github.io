<template>
  <div class="blog-list-container">
    <div class="table-header">
      <span>名称</span>
      <span>创建时间</span>
    </div>
    <ul v-if="!isNoData" class="issue-list" ref="issueList">
      <li class="issue-item" v-for="issue in issues" :key="issue.id">
        <router-link tag="a" :to="{name: 'BlogDetail', params: {number: issue.number, issue: issue}}">
          {{issue.title}}
        </router-link>
        <ul>
          <li class="tag tag-small" v-for="label in issue.labels" :key="label.id" @click="setActiveLabel(label)"
              :style="{ backgroundColor: '#' + label.color}">{{label.name}}
          </li>
        </ul>
        <span class="time">
          {{$moment(issue.created_at).format('YYYY-MM-DD HH:mm')}}
        </span>
      </li>
    </ul>
    <div v-if="isNoData" class="no-data">
      <div></div>
      <span>该分类下还没有文章哦</span>
    </div>
    <div v-show="loading" class="loader-container">
      <div  class="loader-custom loader loader-7">
        <div class="line line1"></div>
        <div class="line line2"></div>
        <div class="line line3"></div>
      </div>
    </div>

    <div class="bottom-bar">
      <input type="text" class="fl query" v-model="keyword" placeholder="按文章标题或内容搜索..."
             @keyup.enter="searchIssues()"/>
      <pagination :totalNum="totalNum" :currentPage="currentPage" :pageSize="pageSize"
                  @currentPageChanged="handleCurrentPageChanged"/>
    </div>
  </div>
</template>
<script>
  import { mapGetters, mapActions } from 'vuex'
  import Pagination from '../components/Pagination.vue'

  export default {
    name: 'blog-list',
    data () {
      return {
        keyword: '',
        totalNum: 0,
        currentPage: 1,
        issues: [],
        isNoData: false,
        loading: false
      }
    },
    components: {Pagination},
    watch: {
      activeLabel () {
        this.keyword = ''
        this.totalNum = 0
        this.currentPage = 1
        this.getIssues()
      }
    },
    computed: {
      ...mapGetters([
        'activeLabel',
        'pageSize'
      ])
    },
    methods: {
      ...mapActions([
        'updateActiveLabel'
      ]),
      setActiveLabel (label) {
        this.updateActiveLabel(label)
      },
      searchIssues () {
        this.currentPage = 1
        this.getIssues()
      },
      handleCurrentPageChanged (val) {
        this.currentPage = val
        this.getIssues()
      },
      getIssues () {
        this.isNoData = false
        this.issues = []
        this.loading = true
        this.$gitHubApi.getIssues(this, {
          label: this.activeLabel ? this.activeLabel.name : '',
          keyword: this.keyword,
          currentPage: this.currentPage,
          pageSize: this.pageSize
        }).then(response => {
          // 加载完数据后滚动到顶部
          if (this.$refs.issueList) {
            this.$refs.issueList.scrollTop = 0
          }
          this.totalNum = response.data.total_count
          this.issues = response.data.items
          if (!this.issues || this.issues.length === 0) {
            this.isNoData = true
          }
          this.loading = false
        }).catch(() => {
          this.loading = false
        })
      }
    },
    mounted () {
      this.$nextTick(() => {
        document.title = '醉后不知天在水'
        this.getIssues()
      })
    }
  }
</script>
<style lang="scss" scoped>
  $time-width: 150px;

  .blog-list-container {
    flex-grow: 1;
    display: flex;
    // overflow-y: auto; // 本来只在 issue-list 上添加 overflow-y: auto 就可以的，在 blog-list-container 里也添加 overflow-y: auto 是为了兼容 Firefox 和 Android
    overflow: hidden;
    flex-direction: column;
  }

  .table-header {
    flex: 0 0 48px;
    display: flex;
    align-items: center;
    background-color: #f9fafc;
    padding: 0px 70px;
    border-top: 1px solid #eeeeee;
    border-bottom: 1px solid #eeeeee;
    font-size: 12px;
    color: #a3b4bc;
    :first-child {
      flex-grow: 1;
    }
    :last-child {
      flex: 0 0 $time-width;
    }
  }

  .issue-list {
    flex-grow: 1;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    list-style: none;
    .issue-item {
      flex: 0 0 69px;
      border-bottom: solid 1px #eeeeee;
      display: flex;
      align-items: center;
      padding: 0px 70px;
      a {
        font-size: 15px;
        color: #4b595f;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
      }
      a:hover {
        color: #3593f2;
      }
      ul {
        display: flex;
        flex: 1 0 auto;
        margin: 0 30px;
        list-style: none;
      }
      .time {
        font-size: 15px;
        color: #4b595f;
        cursor: default;
        flex: 0 0 $time-width;
      }
    }
  }

  .no-data {
    flex-grow: 1;
    overflow-y: auto;
    display: flex;
    flex-direction: column;

    div {
      flex: 1 0 160px;
      background: url("../assets/no-data.png") no-repeat bottom;
    }
    span {
      flex-grow: 1;
      text-align: center;
      padding: 30px;
      color: #666666;
      font-size: 14px;
    }
  }
  .loader-container {
    flex-grow: 1;  
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .bottom-bar {
    flex: 0 0 69px;
    padding: 0px 40px;
    border-top: 1px solid #eeeeee;
    display: flex;
    align-items: center;
    .query {
      flex-grow: 1;
      height: 16px;
      border-radius: 3px;
      padding: 15px 30px 15px 15px;
      border: solid 1px #eeeeee;
      font-size: 14px;
      color: #4b595f;
      margin-right: 20px;
      outline: none;
      background: #f9fafc url("../assets/enter-icon.svg") no-repeat calc(100% - 10px) center;
    }

    .query::placeholder {
      color: #849aa4;
    }
  }

</style>
