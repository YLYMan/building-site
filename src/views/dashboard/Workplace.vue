<template>
  <div>
    <a-row :gutter="24">
      <a-col :xl="7" :lg="24" :md="24" :sm="24" :xs="24">
        <a-card
          class="project-list"
          :loading="loading"
          style="margin-bottom: 24px;"
          :bordered="false"
          title="项目概要"
        >
          <!-- <a slot="extra">全部项目</a> -->
          <div class="list-box">
            <div>项目类型：xxx</div>
            <div>总投资金额：xxx</div>
            <div>建筑面积：xxx</div>
            <div>建设单位：xxx</div>
            <div>监理单位：xxx</div>
            <div>勘察单位：xxx</div>
            <div>施工单位：xxx</div>
            <div>设计单位：xxx</div>
          </div>
        </a-card>

        <a-card :loading="loading" title="人员分析" :bordered="false">
          <div class="extra-content">
            <div class="stat-item">
              <a-statistic title="场内人数" :value="56" class="statis-style" />
            </div>
            <div class="stat-item">
              <a-statistic title="劳务人数" :value="8" class="statis-style" />
            </div>
            <div class="stat-item">
              <a-statistic title="管理人员" :value="2223" class="statis-style" />
            </div>
          </div>
        </a-card>
      </a-col>
      <a-col
        style="padding: 0 12px"
        :xl="11"
        :lg="24"
        :md="24"
        :sm="24"
        :xs="24">
        <a-card
          title=""
          style="margin-bottom: 24px"
          :loading="radarLoading"
          :bordered="false"
          :body-style="{ padding: 0 }"
        >
          <div style="min-height: 300px;">
            <!-- :scale="scale" :axis1Opts="axis1Opts" :axis2Opts="axis2Opts"  -->
            <!-- <radar :data="radarData" /> -->
            <a-carousel autoplay>
              <div><h3>1</h3></div>
              <div><h3>2</h3></div>
              <div><h3>3</h3></div>
            </a-carousel>
          </div>
        </a-card>
        <a-card :loading="loading" title="里程碑节点" :bordered="false">
          <!-- <div class="members">
            <a-row>
              <a-col :span="12" v-for="(item, index) in teams" :key="index">
                <a>
                  <a-avatar size="small" :src="item.avatar" />
                  <span class="member">{{ item.name }}</span>
                </a>
              </a-col>
            </a-row>
          </div> -->
          <div slot="extra">
            计划开工日期： 2022-01-11
            计划竣工日期： 2022-11-12
          </div>
          <div class="node-box">
            <div class="node-l">
              <a-statistic title="已施工(天)" :value="12" style="text-align: center;" />
            </div>
            <div class="node-c">
              <!-- <a-timeline>
                <a-timeline-item>Create a services site</a-timeline-item>
                <a-timeline-item>Solve initial network problems</a-timeline-item>
                <a-timeline-item>Technical testing</a-timeline-item>
              </a-timeline> -->

              <a-steps progress-dot :current="1">
                <a-step title="Finished" description="" />
                <a-step title="In Progress" description="" />
                <a-step title="Waiting" description="" />
              </a-steps>
            </div>
            <div class="node-r">
              <a-statistic title="总工期(天)" :value="12" style="text-align: center;" />
            </div>
          </div>
        </a-card>
      </a-col>
      <a-col
        :xl="6"
        :lg="24"
        :md="24"
        :sm="24"
        :xs="24">
        <a-card
          title="天气情况"
          style="margin-bottom: 24px"
          :bordered="false"
          :body-style="{ padding: 0 }"
        >
          <a slot="extra">太原</a>
          <div class="item-group">
            <ul>
              <!-- <li>
                <div>{{ weatherInfo.now.text }}</div> <div>{{ weatherInfo.now.temperature }} ℃</div>
              </li>
              <li>
                <div>风向</div>
                <div>级</div>
              </li>
              <li>
                <div>湿度</div>
                <div>%</div>
              </li> -->
              <li>
                <div><img :src="'/weatherIcon/weathercn/' + weatherInfo.img + '.png'" alt=""></div> <div>{{ weatherInfo.temp }} ℃</div>
              </li>
              <li>
                <div>风向</div>
                <div>{{ weatherInfo.windpower }} {{ weatherInfo.winddirect }}</div>
              </li>
              <li>
                <div>湿度</div>
                <div>{{ weatherInfo.humidity }}%</div>
              </li>
              <li>
                <div>PM25</div>
                <div>{{ weatherInfo.aqi.pm2_5 }}μg/m3</div>
              </li>
              <li>
                <div>PM10</div>
                <div>{{ weatherInfo.aqi.pm10 }}μg/m3</div>
              </li>
            </ul>
          </div>
        </a-card>
        <a-card
          title="设备统计"
          style="margin-bottom: 24px"
          :loading="radarLoading"
          :bordered="false"
          :body-style="{ padding: 0 }"
        >
          <div style="min-height: 300px;">
            <!-- :scale="scale" :axis1Opts="axis1Opts" :axis2Opts="axis2Opts"  -->
            <!-- <radar :data="radarData" /> -->
            <a-table :columns="columns" :data-source="data"></a-table>
          </div>
        </a-card>
      </a-col>
    </a-row>
  </div>
</template>

<script>
import { timeFix } from '@/utils/util'
import { mapState } from 'vuex'
import { PageHeaderWrapper } from '@ant-design-vue/pro-layout'
// import { Radar } from '@/components'

import { getRoleList, getServiceList } from '@/api/manage'

const DataSet = require('@antv/data-set')

const city = '太原'
const weatherSuccses = '10000'

export default {
  name: 'Workplace',
  components: {
    PageHeaderWrapper
    // Radar
  },
  data () {
    return {
      timeFix: timeFix(),
      avatar: '',
      user: {},

      projects: [],
      loading: true,
      radarLoading: true,
      activities: [],
      teams: [],

      // data
      axis1Opts: {
        dataKey: 'item',
        line: null,
        tickLine: null,
        grid: {
          lineStyle: {
            lineDash: null
          },
          hideFirstLine: false
        }
      },
      axis2Opts: {
        dataKey: 'score',
        line: null,
        tickLine: null,
        grid: {
          type: 'polygon',
          lineStyle: {
            lineDash: null
          }
        }
      },
      scale: [
        {
          dataKey: 'score',
          min: 0,
          max: 80
        }
      ],
      axisData: [
        { item: '引用', a: 70, b: 30, c: 40 },
        { item: '口碑', a: 60, b: 70, c: 40 },
        { item: '产量', a: 50, b: 60, c: 40 },
        { item: '贡献', a: 40, b: 50, c: 40 },
        { item: '热度', a: 60, b: 70, c: 40 },
        { item: '引用', a: 70, b: 50, c: 40 }
      ],
      radarData: [],
      columns: [
        {
          title: '设备名称',
          dataIndex: 'name',
          key: 'name'
        },
        {
          title: '设备数量',
          dataIndex: 'num',
          key: 'num'
        },
        {
          title: '状态(在线/离线)',
          dataIndex: 'status',
          key: 'status'
        }
      ],
      data: [
        {
          key: '1',
          name: '视频监控',
          num: 32,
          status: '7/8'
        }
      ],
      weatherInfo: {} // 天气
    }
  },
  computed: {
    ...mapState({
      nickname: state => state.user.nickname,
      welcome: state => state.user.welcome
    }),
    currentUser () {
      return {
        name: 'Serati Ma',
        avatar: 'https://gw.alipayobjects.com/zos/antfincdn/XAosXuNZyF/BiazfanxmamNRoxxVxka.png'
      }
    },
    userInfo () {
      return this.$store.getters.userInfo
    }
  },
  created () {
    this.user = this.userInfo
    this.avatar = this.userInfo.avatar

    getRoleList().then(res => {
      // console.log('workplace -> call getRoleList()', res)
    })

    getServiceList().then(res => {
      // console.log('workplace -> call getServiceList()', res)
    })
  },
  mounted () {
    this.getProjects()
    this.getActivity()
    this.getTeams()
    this.initRadar()
    this.getWeather()
  },
  methods: {
    getWeather() {
      this.$http.get('/weather' + '?city=' + city + '&' + '&appkey=' + 'd2660b1d1f1dcf98cbb58c0a9ac6a7e6').then(res => {
        console.log(res)
        if (res.code === weatherSuccses) {
          this.weatherInfo = res.result.result
          console.log(this.weatherInfo)
        }
      })
    },
    getProjects () {
      this.$http.get('/list/search/projects').then(res => {
        this.projects = res.result && res.result.data
        this.loading = false
      })
    },
    getActivity () {
      this.$http.get('/workplace/activity').then(res => {
        this.activities = res.result
      })
    },
    getTeams () {
      this.$http.get('/workplace/teams').then(res => {
        this.teams = res.result
      })
    },
    initRadar () {
      this.radarLoading = true

      this.$http.get('/workplace/radar').then(res => {
        const dv = new DataSet.View().source(res.result)
        dv.transform({
          type: 'fold',
          fields: ['个人', '团队', '部门'],
          key: 'user',
          value: 'score'
        })

        this.radarData = dv.rows
        this.radarLoading = false
      })
    }
  }
}
</script>

<style lang="less">
@import './Workplace.less';

.ant-table-thead > tr > th, .ant-table-tbody > tr > td {
  padding: 16px 10px;
}

.project-list {
  .list-box {
    div {
      margin-bottom: 10px;
    }
  }
}

.item-group {
  padding: 20px 20px 8px;
  // font-size: 0;

  // a {
  //   color: rgba(0, 0, 0, 0.65);
  //   display: inline-block;
  //   font-size: 14px;
  //   margin-bottom: 13px;
  //   width: 25%;
  // }
  ul {
    padding-left: 0;
    li {
      display: flex;
      justify-content: space-around;
      margin-bottom: 6px;
      &:first-child {
        margin-bottom: 12px;
        font-size: 26px;
        font-weight: bold;
        color: #666666;
      }
      div {
        min-width: 100px;
        &:last-child {
          display: flex;
          align-items: center;
          justify-content: center;
        }
      }
    }
  }
}

.members {
  a {
    display: block;
    margin: 12px 0;
    line-height: 24px;
    height: 24px;

    .member {
      font-size: 14px;
      color: rgba(0, 0, 0, 0.65);
      line-height: 24px;
      max-width: 100px;
      vertical-align: top;
      margin-left: 12px;
      transition: all 0.3s;
      display: inline-block;
    }

    &:hover {
      span {
        color: #1890ff;
      }
    }
  }
}

.mobile {
  .project-list {
    .project-card-grid {
      width: 100%;
    }
  }

  .more-info {
    border: 0;
    padding-top: 16px;
    margin: 16px 0 16px;
  }

  .headerContent .title .welcome-text {
    display: none;
  }
}
</style>
