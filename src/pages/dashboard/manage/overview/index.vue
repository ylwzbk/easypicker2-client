<template>
  <div class="overview">
    <div class="card-list">
      <div v-for="c in cardList" :key="c.type" class="card">
        <div class="logo">
          <el-icon :color="c.color">
            <component :is="c.icon"></component>
          </el-icon>
        </div>
        <div class="content">
          <div class="title">{{ c.title }}</div>
          <div class="text">{{ c.value }}</div>
          <div class="supplement">{{ c.supplement }}</div>
        </div>
      </div>
    </div>
    <div class="panel">
      <div class="p10 log-filter">
        <span class="item">
          <span class="label">类型</span>
          <el-select v-model="filterLogType" size="default" placeholder="请选择日志类型">
            <el-option
              v-for="(item, idx) in logTypeList"
              :key="idx"
              :label="item.label"
              :value="item.type"
            ></el-option>
          </el-select>
        </span>
        <span class="item">
          <el-input
            size="default"
            clearable
            placeholder="请输入要检索的内容"
            :prefix-icon="Search"
            v-model="searchWord"
          ></el-input>
        </span>
        <span class="item">
          <el-button size="default" :icon="Refresh" @click="refreshLogs">刷新</el-button>
        </span>
        <span class="item">
          <el-button
            size="default"
            type="primary"
            :icon="DataAnalysis"
            @click="exportLogData"
          >导出日志 {{ filterLogs.length }} 条</el-button>
        </span>
      </div>
      <el-table
        tooltip-effect="dark"
        height="400"
        stripe
        border
        :default-sort="{ prop: 'date', order: 'descending' }"
        :data="filterLogs"
        style="width: 100%;"
      >
        <el-table-column sortable prop="date" label="日期" width="180">
          <template #default="scope">{{ formatDate(new Date(scope.row.date)) }}</template>
        </el-table-column>
        <!-- <el-table-column prop="type" label="类型" width="100">
          <template #default="scope">{{ getLogsTypeText(scope.row.type) }}</template>
        </el-table-column>-->
        <el-table-column sortable prop="ip" label="IP" width="100"></el-table-column>
        <el-table-column min-width="160" prop="msg" label="内容"></el-table-column>
        <el-table-column fixed="right" label="操作" width="100">
          <template #default="scope">
            <el-button @click="handleDetail(scope.row.id)" type="text" size="small">查看详情</el-button>
          </template>
        </el-table-column>
      </el-table>

      <div class="flex fc p10">
        <el-pagination
          :current-page="pageCurrent"
          @current-change="handlePageChange"
          background
          :page-count="pageCount"
          :page-sizes="[10, 50, 100, 200, 500, 1000]"
          :page-size="pageSize"
          @size-change="handleSizeChange"
          :total="logSumCount"
          layout="total, sizes, prev, pager, next, jumper"
        ></el-pagination>
      </div>
    </div>
    <el-dialog v-model="showDetail" title="详细信息" width="50%" center :fullscreen="isMobile">
      <!-- TODO: 展示优化 -->
      <!-- <pre style="overflow: hidden;">{{ showData }}</pre> -->
      <json-viewer :value="jsonData" :expand-depth="5" copyable boxed sort></json-viewer>
      <template #footer>
        <span class="dialog-footer">
          <el-button type="default" @click="handleCopyDetail">复制</el-button>
          <el-button type="primary" @click="showDetail = false">确定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>
<script lang="ts" setup>
// import { ElMessage } from 'element-plus'
import {
  computed, onMounted, reactive, ref, watchEffect,
} from 'vue'
import {
  User, Document, Tickets, DataBoard, Search, Refresh, DataAnalysis,
} from '@element-plus/icons-vue'
import { useStore } from 'vuex'
import { ElMessage } from 'element-plus'
import { SuperOverviewApi } from '@/apis'
import { copyRes, formatDate } from '@/utils/stringUtil'
import { tableToExcel } from '@/utils/networkUtil'

const $store = useStore()

const isMobile = computed(() => $store.getters['public/isMobile'])

const cardList = reactive([
  {
    type: 'user',
    title: '用户数量',
    value: '0',
    supplement: '较昨日 +10',
    icon: User,
    color: '#40c9c6',
  },
  {
    type: 'file',
    title: '文件数量',
    value: '0',
    supplement: '较昨日 +10',
    icon: Document,
    color: '#36a3f7',
  },
  {
    type: 'log',
    title: '日志数量',
    value: '0',
    supplement: '较昨日 +10',
    icon: Tickets,
    color: '#f4516c',
  },
  {
    type: 'pv',
    title: 'PV/UV',
    value: '0/0',
    supplement: '',
    icon: DataBoard,
    color: '#34bfa3',
  },
])
// 刷新记录条数
const refreshCount = () => {
  SuperOverviewApi.getCount().then((res) => {
    const {
      user, file, log, pv,
    } = res.data
    cardList[0].value = `${user.sum}`
    cardList[0].supplement = `较昨日 +${user.recent}`
    cardList[1].value = `${file.sum}`
    cardList[1].supplement = `较昨日 +${file.recent}`
    cardList[2].value = `${log.sum}`
    cardList[2].supplement = `较昨日 +${log.recent}`
    cardList[3].value = `${pv.today.sum}/${pv.today.uv}`
    cardList[3].supplement = `历史: ${pv.all.sum}/${pv.all.uv}`
  })
}

// 日志
const logs: any[] = reactive([])

// function getLogsTypeText(type: string) {
//   const logsTypeText: any = {
//     request: '网络请求',
//     behavior: '用户行为',
//     error: '错误',
//     pv: '页面访问',
//   }
//   return logsTypeText[type]
// }
// 筛选的日志
const filterLogType = ref('behavior')
const searchWord = ref('')
const logTypeList = reactive([
  {
    label: '用户行为',
    type: 'behavior',
  },
  {
    label: '网络请求',
    type: 'request',
  }, {
    label: '服务端错误',
    type: 'error',
  },
  {
    label: '页面访问',
    type: 'pv',
  },
])

const filterLogs = computed(() => logs
  .filter((v) => v.type === filterLogType.value))
// 后端搜索
// .filter((v) => {
//   const { date, ip, msg } = v
//   if (searchWord.value.length === 0) return true
//   return `${date} ${ip} ${msg}`.includes(searchWord.value)
// })
// 分页
// 页大小
const pageSize = ref(10)
const handleSizeChange = (v: number) => {
  pageSize.value = v
}
// 总条数
const logSumCount = ref(0)
const pageCount = computed(() => {
  const t = Math.ceil(logSumCount.value / pageSize.value)
  return t
})
const pageCurrent = ref(1)
// const pageLogs = computed(() => {
//   const start = (pageCurrent.value - 1) * pageSize.value
//   const end = (pageCurrent.value) * pageSize.value
//   return filterLogs.value.slice(start, end)
// })
const handlePageChange = (idx: number) => {
  pageCurrent.value = idx
}

// 最简单防抖与节流相结合
let timer = null
let time = Date.now()
let lock = false
const wait = 500
const refreshLogs = () => {
  const callback = () => {
    lock = true
    SuperOverviewApi.getLogMsg(pageSize.value, pageCurrent.value, filterLogType.value, searchWord.value).then((res) => {
      logs.splice(0, logs.length)
      logs.push(...res.data.logs)
      logSumCount.value = res.data.sum
      lock = false
    })
  }
  if (Date.now() > wait + time && !lock) {
    time = Date.now()
    timer = null
  }
  if (timer) {
    clearTimeout(timer)
    timer = setTimeout(callback, wait)
  } else {
    callback()
    timer = 1
  }
}
watchEffect(() => {
  if (filterLogType.value) {
    pageCurrent.value = 1
  }
})

watchEffect(() => {
  if (searchWord.value !== undefined) {
    refreshLogs()
  }
})

watchEffect(() => {
  if (pageCurrent.value || filterLogType.value || pageSize.value) {
    refreshLogs()
  }
})
const showDetail = ref(false)
const showData = ref('')
const handleDetail = (id) => {
  SuperOverviewApi.getLogMsgDetail(id).then((res) => {
    showDetail.value = true
    showData.value = JSON.stringify(res.data, null, 2)
  })
}
const jsonData = computed(() => {
  try {
    return JSON.parse(showData.value)
  } catch (e) {
    return {}
  }
})
const handleCopyDetail = () => {
  copyRes(showData.value)
}

const exportLogData = () => {
  if (filterLogs.value.length === 0) {
    return
  }
  const headers = ['日期', 'IP', '内容']
  const body = filterLogs.value.map((v) => {
    const { date, ip, msg } = v
    return [formatDate(new Date(date)), ip, msg]
  })
  tableToExcel(headers, body, `导出日志_${filterLogs.value.length}条${formatDate(new Date(), 'yyyy年MM月日hh时mm分ss秒')}.xls`)
  ElMessage.success('导出成功')
}
onMounted(() => {
  refreshCount()
  refreshLogs()
})

</script>

<style scoped lang="scss">
@media screen and (max-width: 700px) {
  .card-list {
    margin-top: 40px;
  }
  .card {
    min-width: 300px;
  }
  .log-filter {
    justify-content: center;
  }
}
.overview {
  margin: 0 auto;
}
.card-list {
  display: flex;
  margin-top: 20px;
  justify-content: center;
  flex-wrap: wrap;
}
.card {
  margin: 10px;
  height: 108px;
  cursor: pointer;
  font-size: 12px;
  position: relative;
  overflow: hidden;
  color: #666;
  background: #fff;
  box-shadow: 4px 4px 40px rgb(0 0 0 / 5%);
  border-color: rgba(0, 0, 0, 0.05);
  width: 260px;
  .logo {
    float: left;
    margin: 16px 10px 0 10px;
    -webkit-transition: all 0.38s ease-out;
    transition: all 0.38s ease-out;
    border-radius: 6px;
    font-size: 48px;
    i {
      padding: 10px;
    }
  }
  .content {
    float: right;
    font-weight: 700;
    margin: 26px;
    margin-left: 0;
    .title {
      line-height: 18px;
      color: rgba(0, 0, 0, 0.45);
      font-size: 16px;
    }
    .text {
      font-size: 20px;
    }
    .supplement {
      font-size: 12px;
      font-weight: lighter;
    }
  }
}
.panel {
  max-width: 1024px;
  padding: 1em;
  background-color: #fff;
  margin: 10px auto;
  box-sizing: border-box;
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
  border-radius: 4px;
}
.log-filter {
  display: flex;
  flex-wrap: wrap;
  .item {
    margin-right: 10px;
    margin-bottom: 10px;
    .label {
      margin-right: 10px;
      font-size: 12px;
    }
  }
}
</style>
