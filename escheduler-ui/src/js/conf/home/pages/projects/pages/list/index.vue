<template>
  <m-list-construction :title="$t('Project')">
    <template slot="conditions">
      <m-conditions @on-conditions="_onConditions">
        <template slot="button-group">
          <x-button type="ghost" size="small" @click="_create('')">{{$t('Create Project')}}</x-button>
        </template>
      </m-conditions>
    </template>
    <template slot="content">
      <template v-if="projectsList.length">
        <m-list :projects-list="projectsList" @on-update="_onUpdate" :page-no="searchParams.pageNo" :page-size="searchParams.pageSize"></m-list>
        <div class="page-box">
          <x-page :current="parseInt(searchParams.pageNo)" :total="total" :page-size="searchParams.pageSize" show-elevator @on-change="_page"></x-page>
        </div>
      </template>
      <template v-if="!projectsList.length">
        <m-no-data></m-no-data>
      </template>
      <m-spin :is-spin="isLoading" :is-left="false"></m-spin>
    </template>
  </m-list-construction>
</template>
<script>
  import _ from 'lodash'
  import { mapActions } from 'vuex'
  import mList from './_source/list'
  import mSpin from '@/module/components/spin/spin'
  import mCreateProject from './_source/createProject'
  import mNoData from '@/module/components/noData/noData'
  import listUrlParamHandle from '@/module/mixin/listUrlParamHandle'
  import mConditions from '@/module/components/conditions/conditions'
  import mListConstruction from '@/module/components/listConstruction/listConstruction'


  export default {
    name: 'projects-list',
    data () {
      return {
        total: null,
        projectsList: [],
        isLoading: true,
        searchParams: {
          pageSize: 10,
          pageNo: 1,
          searchVal: ''
        }
      }
    },
    mixins: [listUrlParamHandle],
    props: {},
    methods: {
      ...mapActions('projects', ['getProjectsList']),
      /**
       * Inquire
       */
      _onConditions (o) {
        this.searchParams = _.assign(this.searchParams, o)
        this.searchParams.pageNo = 1
      },

      _page (val) {
        this.searchParams.pageNo = val
      },
      _create (item) {
        let self = this
        let modal = this.$modal.dialog({
          closable: false,
          showMask: true,
          escClose: true,
          className: 'v-modal-custom',
          transitionName: 'opacityp',
          render (h) {
            return h(mCreateProject, {
              on: {
                onUpdate () {
                  self._debounceGET()
                  modal.remove()
                }
              },
              props: {
                item: item
              }
            })
          }
        })
      },
      _onUpdate () {
        this._debounceGET()
      },
      _getList (flag) {
        this.isLoading = !flag
        this.getProjectsList(this.searchParams).then(res => {
          this.projectsList = []
          this.projectsList = res.totalList
          this.total = res.total
          this.isLoading = false
        }).catch(e => {
          this.isLoading = false
        })
      }
    },
    watch: {
      // router
      '$route' (a) {
        // url no params get instance list
        this.searchParams.pageNo = _.isEmpty(a.query) ? 1 : a.query.pageNo
      }
    },
    created () {
    },
    mounted () {
    },
    components: { mListConstruction, mSpin, mConditions, mList, mCreateProject, mNoData }
  }
</script>
