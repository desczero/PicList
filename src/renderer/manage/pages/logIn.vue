<template>
  <div class="layout">
    <el-tabs
      v-model="activeName"
      type="border-card"
      stretch
      style="height: calc(100vh - 50px);width: 100%;overflow-x: hidden;"
      tab-position="left"
      @tab-change="getExistingConfig(activeName)"
    >
      <el-tab-pane
        name="login"
        :label="$T('MANAGE_LOGIN_PAGE_PANE_NAME')"
        style="width: 100%;overflow-y: scroll;height: calc(100vh - 50px);"
      >
        <el-alert
          v-loading="isLoading"
          :title="$T('MANAGE_LOGIN_PAGE_PANE_DESC')"
          type="success"
          show-icon
          center
          :element-loading-text="$T('MANAGE_LOGIN_PAGE_PANE_LOADING')"
          :element-loading-spinner="svg"
          element-loading-svg-view-box="0, 0, 150, 150"
        />
        <el-row>
          <el-col
            v-for="item in sortedAllConfigAliasMap"
            :key="item"
            :xs="24"
            :sm="12"
            :md="8"
            :lg="6"
            :xl="4"
          >
            <el-card
              class="box-card"
              style="margin: 10px 0;"
              shadow="hover"
            >
              <el-popover
                placement="top"
                :width="300"
                trigger="click"
              >
                <el-table
                  :data="formObjToTableData(item.config)"
                  style="width: 100%"
                  size="small"
                  :header-cell-style="{'text-align':'center'}"
                  :cell-style="{'text-align':'center'}"
                >
                  <el-table-column
                    prop="key"
                    :label="$T('MANAGE_LOGIN_PAGE_PANE_KEY_NAME')"
                    width="100"
                  />
                  <el-table-column
                    prop="value"
                    :label="$T('MANAGE_LOGIN_PAGE_PANE_KEY_VALUE')"
                  />
                </el-table>
                <template #reference>
                  <el-button
                    style="width: 100%; text-align: center;overflow: hidden;text-overflow: ellipsis;white-space: nowrap;"
                  >
                    <template #icon>
                      <img
                        :src="require(`./assets/${item.picBedName}.webp`)"
                        style="width: 25px; height: 25px;"
                      >
                    </template>
                    {{ item.alias }}
                  </el-button>
                </template>
              </el-popover>
              <br>
              <br>
              <el-button-group>
                <el-button
                  type="primary"
                  :icon="Pointer"
                  plain
                  @click="handleConfigClick(item)"
                >
                  {{ $T('MANAGE_LOGIN_PAGE_PANE_ENTER') }}
                </el-button>
                <el-button
                  type="warning"
                  :icon="Delete"
                  plain
                  @click="handleConfigRemove(item.alias)"
                >
                  {{ $T('MANAGE_LOGIN_PAGE_PANE_DELETE') }}
                </el-button>
              </el-button-group>
            </el-card>
          </el-col>
        </el-row>
      </el-tab-pane>
      <el-tab-pane
        v-for="item in supportedPicBedList"
        :key="item.name"
        :label="item.name"
        :name="item.icon"
        class="tab-pane"
        lazy
        style="width: 100%;overflow-y: scroll;height: calc(100vh - 50px);"
      >
        <el-alert
          :title="item.explain"
          type="info"
          show-icon
          center
          :closable="false"
        />
        <el-alert
          center
          :closable="false"
        >
          <div>
            {{ item.referenceText }} <a
              style="color:blue;cursor:pointer"
              @click="handleReferenceClick(item.refLink)"
            >{{ item.refLink }}</a>
          </div>
        </el-alert>
        <el-form
          label-position="top"
          require-asterisk-position="right"
          label-width="10vw"
          size="default"
          :rules="rules"
          :model="configResult"
        >
          <el-form-item
            v-for="option in supportedPicBedList[item.icon].options"
            :key="option"
            :prop="item.icon + '.' + option"
          >
            <template #label>
              {{ supportedPicBedList[item.icon].configOptions[option].description }}
              <el-tooltip
                v-if="!!supportedPicBedList[item.icon].configOptions[option].tooltip"
                effect="dark"
                :content="supportedPicBedList[item.icon].configOptions[option].tooltip"
                placement="right"
              >
                <el-icon
                  color="#409EFF"
                >
                  <InfoFilled />
                </el-icon>
              </el-tooltip>
            </template>
            <el-input
              v-if="supportedPicBedList[item.icon].configOptions[option].type === 'string'"
              v-model.trim="configResult[item.icon + '.' + option]"
              :placeholder="supportedPicBedList[item.icon].configOptions[option].placeholder"
              :disabled="!!supportedPicBedList[item.icon].configOptions[option].disabled"
            />
            <el-switch
              v-else-if="supportedPicBedList[item.icon].configOptions[option].type === 'boolean'"
              v-model="configResult[item.icon + '.' + option]"
              active-color="#13ce66"
              inactive-color="#ff4949"
            />
            <el-input
              v-else-if="supportedPicBedList[item.icon].configOptions[option].type === 'number'"
              v-model.number="configResult[item.icon + '.' + option]"
              :placeholder="supportedPicBedList[item.icon].configOptions[option].placeholder"
            />
            <el-select
              v-else-if="supportedPicBedList[item.icon].configOptions[option].type === 'select'"
              v-model="configResult[item.icon + '.' + option]"
              :placeholder="$T('MANAGE_LOGIN_PAGE_PANE_SELECT_PLACEHOLDER')"
            >
              <el-option
                v-for="i in Object.entries(supportedPicBedList[item.icon].configOptions[option].selectOptions)"
                :key="i[0]"
                :label="i[1] as string"
                :value="i[0]"
              />
            </el-select>
          </el-form-item>
        </el-form>
        <div style="margin: 0 auto;position: relative;left: 10%;right: 50%;">
          <el-dropdown
            split-button
            type="success"
            style="margin-left: 10vw"
            placement="top"
            :disabled="currentAliasList.length === 0"
          >
            {{ $T('MANAGE_LOGIN_PAGE_PANE_IMPORT') }}
            <template #dropdown>
              <el-dropdown-item
                v-for="i in currentAliasList"
                :key="i"
                @click="handleConfigImport(i)"
              >
                {{ i }}
              </el-dropdown-item>
            </template>
          </el-dropdown>
          <el-button
            type="primary"
            style="margin-left: 10vw"
            :icon="Edit"
            plain
            @click="handleConfigChange(item.icon)"
          >
            {{ $T('MANAGE_LOGIN_PAGE_PANE_SAVE') }}
          </el-button>
          <el-button
            type="danger"
            style="margin-left: 10vw"
            :icon="Delete"
            plain
            @click="handleConfigReset(item.icon)"
          >
            {{ $T('MANAGE_LOGIN_PAGE_PANE_RESET') }}
          </el-button>
        </div>
        <br>
        <el-alert
          :title="$T('MANAGE_LOGIN_PAGE_PANE_TABLE_TITLE')"
          type="success"
          center
          :closable="false"
        />
        <el-table
          :data="dataForTable"
          style="width: 100%;margin-top: 10px"
          :header-cell-style="{'text-align':'center'}"
          :cell-style="{'text-align':'center'}"
          @cell-click="handleCellClick"
        >
          <el-table-column
            v-for="option in supportedPicBedList[item.icon].options"
            :key="option"
            :prop="option"
            :label="supportedPicBedList[item.icon].configOptions[option].description"
            sortable
            show-overflow-tooltip
          />
        </el-table>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script lang="ts" setup>
import { reactive, ref, onMounted, computed } from 'vue'
import { supportedPicBedList } from '../utils/constants'
import { Delete, Edit, Pointer, InfoFilled } from '@element-plus/icons-vue'
import { ElMessage, ElNotification } from 'element-plus'
import { getConfig, saveConfig, removeConfig } from '../utils/dataSender'
import { shell } from 'electron'
import { useRouter } from 'vue-router'
import { useManageStore } from '../store/manageStore'
import { formObjToTableData, svg } from '../utils/common'
import { getConfig as getPicBedsConfig } from '@/utils/dataSender'
import { formatEndpoint } from '~/main/manage/utils/common'
import { T as $T } from '@/i18n'

const activeName = ref('login')
const configResult:IStringKeyMap = reactive({})
const existingConfiguration = reactive({} as IStringKeyMap)
const dataForTable = reactive([] as any[])
const allConfigAliasMap = reactive({} as IStringKeyMap)
const router = useRouter()
const manageStore = useManageStore()
const isLoading = ref(false)

const sortedAllConfigAliasMap = computed(() => {
  const sorted = Object.values(allConfigAliasMap).sort((a, b) => {
    return a.picBedName.localeCompare(b.picBedName)
  })
  return sorted
})

const currentAliasList = reactive([] as string[])

const ruleMap = (options: IStringKeyMap) => {
  const rule:any = {}
  Object.keys(options).forEach((key) => {
    const item = options[key].options
    item.forEach((option: string) => {
      const keyName = key + '.' + option
      if (options[key].configOptions[option].rule) {
        rule[keyName] = options[key].configOptions[option].rule
      }
      if (options[key].configOptions[option].default) {
        configResult[keyName] = options[key].configOptions[option].default
      }
    })
  })
  return rule
}

const rules = ruleMap(supportedPicBedList)

const getDataForTable = () => {
  for (const key in existingConfiguration) {
    const obj = {} as IStringKeyMap
    for (const option in existingConfiguration[key]) {
      obj[option] = existingConfiguration[key][option]
    }
    dataForTable.push(obj)
  }
}

const getExistingConfig = async (name:string) => {
  if (name === 'login') {
    getAllConfigAliasArray()
    return
  }
  currentAliasList.length = 0
  const result = await getConfig<any>('picBed')
  if (!result) {
    existingConfiguration[name] = { fail: '暂无配置' }
  }
  for (const key in existingConfiguration) {
    delete existingConfiguration[key]
  }
  for (const key in result) {
    if (result[key].picBedName === name) {
      existingConfiguration[key] = result[key]
      currentAliasList.push(key)
    }
  }
  dataForTable.length = 0
  getDataForTable()
  handleConfigImport(currentAliasList[0])
}

const getAliasList = () => {
  const aliasList = [] as string[]
  for (const key in existingConfiguration) {
    aliasList.push(existingConfiguration[key].alias)
  }
  return aliasList
}

const handleConfigChange = async (name: string) => {
  const aliasList = getAliasList()
  const allKeys = Object.keys(supportedPicBedList[name].configOptions)
  const resultMap:IStringKeyMap = {}
  const reg = /^[\u4e00-\u9fa5_a-zA-Z0-9-]+$/
  for (const key of allKeys) {
    const resultKey = name + '.' + key
    if (supportedPicBedList[name].configOptions[key].required) {
      if (supportedPicBedList[name].configOptions[key].type !== 'boolean' && !configResult[resultKey]) {
        ElMessage.error(`${$T('MANAGE_LOGIN_PAGE_PANE_CONFIG_CHANGE_MESSAGE_A')} ${supportedPicBedList[name].configOptions[key].description}`)
        return
      }
    }
    if (key === 'alias' && configResult[resultKey] !== undefined && !reg.test(configResult[resultKey])) {
      ElMessage.error($T('MANAGE_LOGIN_PAGE_PANE_CONFIG_CHANGE_ALIAS_MESSAGE'))
      return
    }
    if (key === 'itemsPerPage' && configResult[resultKey] !== undefined && (configResult[resultKey] < 20 || configResult[resultKey] > 1000)) {
      ElMessage.error($T('MANAGE_LOGIN_PAGE_PANE_CONFIG_CHANGE_ITEMS_PER_PAGE_MESSAGE'))
      return
    }
    if ((key === 'customUrl') && configResult[resultKey] !== undefined && configResult[resultKey] !== '') {
      if (name !== 'upyun') {
        if (!/^https?:\/\//.test(configResult[resultKey])) {
          ElMessage.error($T('MANAGE_LOGIN_PAGE_PANE_CONFIG_CHANGE_CUSTOM_URL_MESSAGE'))
          return
        }
      }
    }
    if (supportedPicBedList[name].configOptions[key].default !== undefined && configResult[resultKey] === '') {
      resultMap[key] = supportedPicBedList[name].configOptions[key].default
    } else if (configResult[resultKey] === undefined) {
      if (supportedPicBedList[name].configOptions[key].default !== undefined) {
        resultMap[key] = supportedPicBedList[name].configOptions[key].default
      } else {
        resultMap[key] = ''
      }
    } else {
      resultMap[key] = configResult[resultKey]
    }
  }
  resultMap.picBedName = name
  if (resultMap.bucketName !== undefined) {
    resultMap.transformedConfig = {}
    const bucketName = resultMap.bucketName.split(',')
    const baseDir = resultMap.baseDir?.split(',')
    const area = resultMap.area?.split(',')
    const customUrl = resultMap.customUrl?.split(',')
    const operator = resultMap.operator?.split(',')
    const password = resultMap.password?.split(',')
    for (let i = 0; i < bucketName.length; i++) {
      if (bucketName[i]) {
        resultMap.transformedConfig = {
          ...resultMap.transformedConfig,
          [bucketName[i]]: {
            baseDir: baseDir && baseDir[i] ? baseDir[i] : '/',
            area: area && area[i] ? area[i] : '',
            customUrl: customUrl && customUrl[i] ? /^https?:\/\//.test(customUrl[i]) ? customUrl[i] : 'http://' + customUrl[i] : '',
            operator: operator && operator[i] ? operator[i] : '',
            password: password && password[i] ? password[i] : ''
          }
        }
      }
    }
  }
  if (resultMap.transformedConfig) {
    resultMap.transformedConfig = JSON.stringify(resultMap.transformedConfig)
  }
  saveConfig(`picBed.${resultMap.alias}`, resultMap)
  await manageStore.refreshConfig()
  await getExistingConfig(activeName.value)
  dataForTable.length = 0
  getDataForTable()
  if (aliasList.includes(resultMap.alias)) {
    ElNotification(
      {
        title: $T('MANAGE_LOGIN_PAGE_PANE_CONFIG_CHANGE_NOTICE_NAME'),
        message: `${$T('MANAGE_LOGIN_PAGE_PANE_CONFIG_CHANGE_NOTICE_MESSAGE')}${resultMap.alias}`,
        type: 'warning',
        duration: 500,
        customClass: 'notification',
        offset: 100
      }
    )
  } else {
    ElNotification(
      {
        title: $T('MANAGE_LOGIN_PAGE_PANE_CONFIG_CHANGE_NOTICE_NAME'),
        message: `${$T('MANAGE_LOGIN_PAGE_PANE_CONFIG_CHANGE_NOTICE_MESSAGE_B')}${resultMap.alias}`,
        type: 'success',
        duration: 2000,
        customClass: 'notification',
        offset: 100
      }
    )
  }
}

const handleConfigReset = (name: string) => {
  let keys = Object.keys(configResult)
  keys = keys.filter((key) => key.startsWith(name))
  keys.forEach((key) => {
    configResult[key] = supportedPicBedList[name].configOptions[key.split('.')[1]].default || ''
  })
}

const handleConfigRemove = (name: string) => {
  try {
    removeConfig('picBed', name)
    ElNotification(
      {
        title: $T('MANAGE_LOGIN_PAGE_PANE_CONFIG_CHANGE_NOTICE_NAME'),
        message: `${$T('MANAGE_LOGIN_PAGE_PANE_CONFIG_CHANGE_NOTICE_MESSAGE_C')}${name}`,
        type: 'success',
        duration: 2000,
        customClass: 'notification',
        offset: 100,
        position: 'bottom-right'
      }
    )
    manageStore.refreshConfig()
    getAllConfigAliasArray()
  } catch (error) {
    ElNotification(
      {
        title: $T('MANAGE_LOGIN_PAGE_PANE_CONFIG_CHANGE_NOTICE_NAME'),
        message: `${$T('MANAGE_LOGIN_PAGE_PANE_CONFIG_CHANGE_NOTICE_MESSAGE_D')}${name}${$T('MANAGE_LOGIN_PAGE_PANE_CONFIG_CHANGE_NOTICE_MESSAGE_E')}`,
        type: 'error',
        duration: 2000,
        customClass: 'notification',
        offset: 100,
        position: 'bottom-right'
      }
    )
  }
}

const getAllConfigAliasArray = async () => {
  const result = await getConfig<any>('picBed')
  for (const key in allConfigAliasMap) {
    delete allConfigAliasMap[key]
  }
  if (!result) {
    return
  }
  let i = 0
  Object.keys(result).forEach((key) => {
    allConfigAliasMap[i] = {
      alias: result[key].alias,
      picBedName: result[key].picBedName,
      config: result[key]
    }
    i++
  })
}

const handleCellClick = (row:any, column:any) => {
  navigator.clipboard.writeText(row[column.property])
  ElMessage.success(`${$T('MANAGE_LOGIN_PAGE_PANE_CONFIG_CHANGE_COPY_SUCCESS')}${row[column.property]}`)
}

const handleReferenceClick = (url: string) => shell.openExternal(url)

const handleConfigClick = async (item: any) => {
  const alias = item.alias
  const config = JSON.stringify(item.config)
  const picBedName = item.picBedName
  const result = await getConfig<any>('picBed')
  router.push({
    path: '/main-page/manage-main-page',
    query: {
      alias,
      picBedName,
      config,
      allPicBedConfigure: JSON.stringify(result)
    }
  })
}

function handleConfigImport (alias: string) {
  const selectedConfig = existingConfiguration[alias]
  if (selectedConfig) {
    supportedPicBedList[selectedConfig.picBedName].options.forEach((option: any) => {
      if (selectedConfig[option] !== undefined) {
        configResult[selectedConfig.picBedName + '.' + option] = selectedConfig[option]
      }
      if (typeof selectedConfig[option] === 'boolean') {
        configResult[selectedConfig.picBedName + '.' + option] = selectedConfig[option]
      }
    })
  }
}

async function getCurrentConfigList () {
  const configList = await getPicBedsConfig<any>('uploader') ?? {}
  const pbList = ['aliyun', 'tcyun', 'upyun', 'qiniu', 'smms', 'qiniu', 'github', 'webdavplist', 'aws-s3', 'imgur']
  const filteredConfigList = pbList.map((pb) => {
    const config = configList[pb]
    if (config && config.configList.length > 0) {
      config.configList.forEach((item: any) => {
        item.type = pb
      })
      return config
    } else {
      return null
    }
  }).filter((config) => config && config.configList.length > 0)
  await getAllConfigAliasArray()
  const promises: Promise<any>[] = []
  for (const config of filteredConfigList.flatMap((config) => config.configList)) {
    const pb = config.type
    promises.push(transUpToManage(config, pb))
  }
  await Promise.all(promises)
}

function isImported (alias: string) {
  for (const key in allConfigAliasMap) {
    if (allConfigAliasMap[key].alias === alias) {
      return true
    }
  }
  return false
}

async function transUpToManage (config: IUploaderConfigListItem, picBedName: string) {
  let alias: string = ''
  const resultMap: IStringKeyMap = {}
  switch (picBedName) {
    case 'smms':
      alias = `smms-${config._configName ?? 'Default'}-imp`
      if (!config.token || isImported(alias)) {
        return
      }
      resultMap.alias = alias
      resultMap.picBedName = 'smms'
      resultMap.token = config.token
      resultMap.paging = true
      saveConfig(`picBed.${resultMap.alias}`, resultMap)
      break
    case 'aliyun':
      if (!config.accessKeyId || !config.accessKeySecret) {
        return
      }
      resultMap.alias = `aliyun-${config._configName ?? 'Default'}-imp`
      resultMap.picBedName = 'aliyun'
      resultMap.accessKeyId = config.accessKeyId
      resultMap.accessKeySecret = config.accessKeySecret
      resultMap.bucketName = ''
      resultMap.baseDir = '/'
      resultMap.paging = true
      resultMap.itemsPerPage = 50
      resultMap.isAutoCustomUrl = !config.customUrl
      resultMap.transformedConfig = JSON.stringify(config.customUrl
        ? {
          [config.bucket]: {
            customUrl: config.customUrl
          }
        }
        : {})
      resultMap.paging = true
      saveConfig(`picBed.${resultMap.alias}`, resultMap)
      break
    case 'qiniu':
      alias = `qiniu-${config._configName ?? 'Default'}-imp`
      if (!config.accessKey || !config.secretKey || isImported(alias)) {
        return
      }
      resultMap.alias = alias
      resultMap.picBedName = 'qiniu'
      resultMap.accessKey = config.accessKey
      resultMap.secretKey = config.secretKey
      resultMap.bucketName = ''
      resultMap.baseDir = '/'
      resultMap.isAutoCustomUrl = false
      resultMap.transformedConfig = JSON.stringify({ [config.bucket]: config.url })
      resultMap.paging = true
      resultMap.itemsPerPage = 50
      saveConfig(`picBed.${resultMap.alias}`, resultMap)
      break
    case 'tcyun':
      alias = `tcyun-${config._configName ?? 'Default'}-imp`
      if (!config.secretId || !config.secretKey || config.version === 'v4' || isImported(alias)) {
        return
      }
      resultMap.alias = alias
      resultMap.picBedName = 'tcyun'
      resultMap.secretId = config.secretId
      resultMap.secretKey = config.secretKey
      resultMap.bucketName = ''
      resultMap.baseDir = '/'
      resultMap.appId = config.appId
      resultMap.isAutoCustomUrl = !config.customUrl
      resultMap.transformedConfig = JSON.stringify(config.customUrl
        ? {
          [config.bucket]: {
            customUrl: config.customUrl
          }
        }
        : {})
      resultMap.paging = true
      resultMap.itemsPerPage = 50
      saveConfig(`picBed.${resultMap.alias}`, resultMap)
      break
    case 'github':
      alias = `github-${config._configName ?? 'Default'}-imp`
      if (!config.token || isImported(alias)) {
        return
      }
      resultMap.alias = alias
      resultMap.picBedName = 'github'
      resultMap.token = config.token
      resultMap.githubUsername = config.repo.split('/')[0]
      resultMap.customUrl = ''
      resultMap.proxy = ''
      resultMap.paging = true
      resultMap.itemsPerPage = 50
      saveConfig(`picBed.${resultMap.alias}`, resultMap)
      break
    case 'upyun':
      alias = `upyun-${config._configName ?? 'Default'}-imp`
      if (!config.operator || !config.password || isImported(alias)) {
        return
      }
      resultMap.alias = alias
      resultMap.picBedName = 'upyun'
      resultMap.operator = config.operator
      resultMap.password = config.password
      resultMap.bucketName = config.bucket
      resultMap.baseDir = '/'
      resultMap.customUrl = config.url
      resultMap.transformedConfig = JSON.stringify({
        [config.bucket]: {
          customUrl: config.url,
          baseDir: '/',
          area: '',
          operator: config.operator,
          password: config.password
        }
      })
      resultMap.paging = true
      resultMap.itemsPerPage = 50
      saveConfig(`picBed.${resultMap.alias}`, resultMap)
      break
    case 'webdavplist':
      alias = `webdav-${config._configName ?? 'Default'}-imp`
      if (!config.host || isImported(alias)) {
        return
      }
      resultMap.alias = alias
      resultMap.endpoint = formatEndpoint(config.host, config.sslEnabled)
      resultMap.picBedName = 'webdavplist'
      resultMap.username = config.username
      resultMap.password = config.password
      resultMap.bucketName = 'webdav'
      resultMap.baseDir = config.path || '/'
      resultMap.customUrl = config.customUrl || ''
      resultMap.sslEnabled = !!config.sslEnabled
      resultMap.proxy = ''
      resultMap.transformedConfig = JSON.stringify({
        webdav: {
          operator: '',
          password: config.password,
          baseDir: config.path || '/',
          customUrl: config.customUrl || '',
          area: ''
        }
      })
      saveConfig(`picBed.${resultMap.alias}`, resultMap)
      break
    case 'aws-s3':
      alias = `aws-s3-${config._configName ?? 'Default'}-imp`
      if (!config.accessKeyID || !config.secretAccessKey || isImported(alias)) {
        return
      }
      resultMap.alias = alias
      resultMap.picBedName = 's3plist'
      resultMap.accessKeyId = config.accessKeyID
      resultMap.secretAccessKey = config.secretAccessKey
      resultMap.endpoint = config.endpoint || ''
      resultMap.baseDir = '/'
      resultMap.bucketName = ''
      resultMap.paging = true
      resultMap.itemsPerPage = 50
      resultMap.proxy = ''
      resultMap.sslEnabled = config.endpoint ? config.endpoint.startsWith('https') : false
      resultMap.aclForUpload = 'public-read'
      resultMap.s3ForcePathStyle = config.pathStyleAccess
      resultMap.transformedConfig = JSON.stringify(
        config.urlPrefix
          ? {
            [config.bucketName]: {
              customUrl: config.urlPrefix
            }
          }
          : {}
      )
      saveConfig(`picBed.${resultMap.alias}`, resultMap)
      break
    case 'imgur':
      alias = `imgur-${config._configName ?? 'Default'}-imp`
      if (!config.username || !config.accessToken || isImported(alias)) {
        return
      }
      resultMap.alias = alias
      resultMap.picBedName = 'imgur'
      resultMap.imgurUserName = config.username
      resultMap.accessToken = config.accessToken
      resultMap.proxy = config.proxy || ''
      saveConfig(`picBed.${resultMap.alias}`, resultMap)
      break
    default:
      return
  }
  manageStore.refreshConfig()
}

onMounted(async () => {
  await getCurrentConfigList()
  getAllConfigAliasArray()
})

</script>

<style lang='stylus'>
.layout
  background-color #fff
  position absolute
  left 162px
  right 0
  &.el-tabs
    border 0
    border-style none
.loginpage
  &:hover
    background-color: Beige
</style>
