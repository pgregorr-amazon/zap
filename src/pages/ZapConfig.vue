<template>
  <q-page padding class="row justify-center full-height" :class="getuitheme">
    <Transition name="slide-up" mode="out-in" appear>
      <q-card flat class="q-mt-lg q-rounded-borders-xl bg-glass col-10 column">
        <q-scroll-area class="col q-px-xl">
          <div class="q-py-lg">
            <Transition name="slide-up" mode="out-in" appear>
              <div class="column">
                <img
                  v-if="getLogo"
                  height="40"
                  class="q-mx-auto q-my-lg block w-fit-content"
                  :src="getLogo"
                />
              </div>
            </Transition>
            <div class="row justify-center">
              <q-radio
                v-if="loadPreSessionData.length"
                v-model="customConfig"
                checked-icon="task_alt"
                unchecked-icon="panorama_fish_eye"
                val="select"
                label="Generate New Session"
              />
              <q-radio
                v-if="loadPreSessionData.length"
                v-model="customConfig"
                checked-icon="task_alt"
                class="q-ml-xl"
                unchecked-icon="panorama_fish_eye"
                val="load"
                label="Restore Unsaved Session"
              />
            </div>
            <p
              class="text-center"
              v-if="isMultiplePackage && customConfig === 'select'"
            >
              There are multiple packages of ZCL metadata loaded. Please select
              the one you wish to use with this configuration.
            </p>
            <p class="text-center" v-else-if="customConfig === 'load'">
              These are sessions found in the database that were not saved into
              a .zap file. You can select them here, and continue the work with
              the configuration.
            </p>

            <template v-if="customConfig === 'select'">
              <q-table
                title="Zigbee Cluster Library metadata"
                :rows="zclPropertiesRow"
                :columns="newSessionCol"
                row-key="name"
                :pagination="newGenerationPagination"
                hide-bottom
                flat
                :card-style="{ backgroundColor: 'transparent' }"
              >
                <template v-slot:header="props">
                  <q-tr :props="props">
                    <q-th
                      v-for="col in props.cols"
                      :key="col.name"
                      :props="props"
                    >
                      {{ col.label }}
                    </q-th>
                  </q-tr>
                </template>
                <template v-slot:body="props">
                  <q-tr :props="props" class="table_body">
                    <q-td key="select" :props="props">
                      <q-radio
                        v-model="selectedZclPropertiesData"
                        checked-icon="task_alt"
                        unchecked-icon="panorama_fish_eye"
                        :val="props.row"
                      />
                    </q-td>
                    <q-td key="category" :props="props">
                      <div>{{ props.row.category }}</div>
                      <q-tooltip :offset="[5, 5]">
                        {{ props.row.path }}
                      </q-tooltip>
                    </q-td>
                    <q-td key="description" :props="props">
                      <div>{{ props.row.description }}</div>
                      <q-tooltip :offset="[5, 5]">
                        {{ props.row.path }}
                      </q-tooltip>
                    </q-td>
                    <q-td key="version" :props="props">
                      <div>{{ props.row.version }}</div>
                    </q-td>
                    <q-td key="status" :props="props">
                      <div v-if="props.row.hasWarning || props.row.hasError">
                        <q-icon
                          class="cursor-pointer"
                          :name="props.row.hasError ? 'error' : 'warning'"
                          :color="props.row.hasError ? 'red' : 'orange'"
                          size="2.5em"
                          @click="propertyDataDialog[props.row.id] = true"
                        />
                        <q-dialog v-model="propertyDataDialog[props.row.id]">
                          <q-card>
                            <q-card-section>
                              <div class="row items-center">
                                <div class="col-1">
                                  <q-icon
                                    :name="
                                      props.row.hasError ? 'error' : 'warning'
                                    "
                                    :color="
                                      props.row.hasError ? 'red' : 'orange'
                                    "
                                    size="2em"
                                  />
                                </div>
                                <div class="text-h6 col">
                                  {{ props.row.description }}
                                </div>
                                <div class="col-1 text-right">
                                  <q-btn dense flat icon="close" v-close-popup>
                                    <q-tooltip>Close</q-tooltip>
                                  </q-btn>
                                </div>
                              </div>
                              <div v-if="props.row.hasError">
                                <div
                                  class="text-h6"
                                  style="margin-top: 15px; padding-left: 20px"
                                >
                                  Errors
                                </div>
                                <ul>
                                  <li
                                    v-for="(error, index) in props.row.errors"
                                    :key="'error' + index"
                                    style="margin-bottom: 10px"
                                  >
                                    {{ error }}
                                  </li>
                                </ul>
                              </div>
                              <div v-if="props.row.hasWarning">
                                <div
                                  class="text-h6"
                                  style="margin-top: 15px; padding-left: 20px"
                                >
                                  Warnings
                                </div>
                                <ul>
                                  <li
                                    v-for="(warning, index) in props.row
                                      .warnings"
                                    :key="index"
                                    style="margin-bottom: 10px"
                                  >
                                    {{ warning }}
                                  </li>
                                </ul>
                              </div>
                            </q-card-section>
                          </q-card>
                        </q-dialog>
                      </div>
                      <q-icon
                        v-else
                        name="check_circle"
                        color="green"
                        size="2em"
                      />
                    </q-td>
                  </q-tr>
                </template>
              </q-table>
              <q-table
                title="Zap Generation Templates"
                :rows="zclGenRow"
                :columns="newSessionCol"
                row-key="name"
                :pagination="newGenerationPagination"
                hide-bottom
                flat
                :card-style="{ backgroundColor: 'transparent' }"
              >
                <template v-slot:top>
                  <div class="q-table__title q-mr-md">
                    Zap Generation Templates
                  </div>
                  <small>( Please select a package for generation )</small>
                </template>
                <template v-slot:header="props">
                  <q-tr :props="props">
                    <q-th
                      v-for="col in props.cols"
                      :key="col.name"
                      :props="props"
                    >
                      {{ col.label }}
                    </q-th>
                  </q-tr>
                </template>
                <template v-slot:body="props">
                  <q-tr :props="props" class="table_body">
                    <q-td key="select" :props="props">
                      <q-checkbox
                        v-model="selectedZclGenData"
                        :val="props.row.id"
                        data-test="gen-template"
                      />
                    </q-td>
                    <q-td key="category" :props="props">
                      <div>{{ props.row.category }}</div>
                      <q-tooltip :offset="[5, 5]">
                        {{ props.row.path }}
                      </q-tooltip>
                    </q-td>
                    <q-td key="description" :props="props">
                      <div>{{ props.row.description }}</div>
                      <q-tooltip :offset="[5, 5]">
                        {{ props.row.path }}
                      </q-tooltip>
                    </q-td>
                    <q-td key="version" :props="props">
                      <div>{{ props.row.version }}</div>
                    </q-td>
                    <q-td key="status" :props="props">
                      <div v-if="props.row.hasWarning || props.row.hasError">
                        <q-icon
                          class="cursor-pointer"
                          :name="props.row.hasError ? 'error' : 'warning'"
                          :color="props.row.hasError ? 'red' : 'orange'"
                          size="2.5em"
                          @click="genDataDialog[props.row.id] = true"
                        ></q-icon>
                        <q-dialog v-model="genDataDialog[props.row.id]">
                          <q-card>
                            <q-card-section>
                              <div class="row items-center">
                                <div class="col-1">
                                  <q-icon
                                    :name="
                                      props.row.hasError ? 'error' : 'warning'
                                    "
                                    :color="
                                      props.row.hasError ? 'red' : 'orange'
                                    "
                                    size="2em"
                                  />
                                </div>
                                <div class="text-h6 col">
                                  {{ props.row.description }}
                                </div>
                                <div class="col-1 text-right">
                                  <q-btn dense flat icon="close" v-close-popup>
                                    <q-tooltip>Close</q-tooltip>
                                  </q-btn>
                                </div>
                              </div>
                              <div v-if="props.row.hasError">
                                <div
                                  class="text-h6"
                                  style="margin-top: 15px; padding-left: 20px"
                                >
                                  Errors
                                </div>
                                <ul>
                                  <li
                                    v-for="(error, index) in props.row.errors"
                                    :key="'error' + index"
                                    style="margin-bottom: 10px"
                                  >
                                    {{ error }}
                                  </li>
                                </ul>
                              </div>
                              <div v-if="props.row.hasWarning">
                                <div
                                  class="text-h6"
                                  style="margin-top: 15px; padding-left: 20px"
                                >
                                  Warnings
                                </div>
                                <ul>
                                  <li
                                    v-for="(warning, index) in props.row
                                      .warnings"
                                    :key="index"
                                    style="margin-bottom: 10px"
                                  >
                                    {{ warning }}
                                  </li>
                                </ul>
                              </div>
                            </q-card-section>
                          </q-card>
                        </q-dialog>
                      </div>
                      <q-icon
                        v-else
                        name="check_circle"
                        color="green"
                        size="2em"
                      />
                    </q-td>
                  </q-tr>
                </template>
              </q-table>
            </template>
            <template v-else>
              <q-table
                title=""
                :rows="loadPreSessionData"
                :columns="loadPreSessionCol"
                row-key="name"
                :pagination="pagination"
                flat
                :card-style="{ backgroundColor: 'transparent' }"
              >
                <template v-slot:header="props">
                  <q-tr :props="props">
                    <q-th
                      v-for="col in props.cols"
                      :key="col.name"
                      :props="props"
                    >
                      {{ col.label }}
                    </q-th>
                  </q-tr>
                </template>
                <template v-slot:body="props">
                  <q-tr :props="props" class="table_body">
                    <q-td key="select" :props="props">
                      <q-radio
                        v-model="selectedZclSessionData"
                        checked-icon="task_alt"
                        unchecked-icon="panorama_fish_eye"
                        :val="props.row"
                      />
                    </q-td>
                    <q-td key="zclproperty" :props="props">
                      <div>{{ props.row.zclProperty.description }}</div>
                    </q-td>
                    <q-td key="gen template file" :props="props">
                      <div>{{ props.row.genTemplateFile.version }}</div>
                    </q-td>
                    <q-td key="creation time" :props="props">
                      <div>
                        {{ new Date(props.row.creationTime).toDateString() }}
                      </div>
                    </q-td>
                  </q-tr>
                </template>
              </q-table>
            </template>

            <div class="row justify-center q-mt-xl">
              <q-btn
                color="primary"
                @click="submitForm"
                label="Submit"
                data-test="login-submit"
              />
            </div>
          </div>
        </q-scroll-area>
      </q-card>
    </Transition>
  </q-page>
</template>

<script>
import restApi from '../../src-shared/rest-api.js'
import { QSpinnerGears } from 'quasar'
import { setCssVar } from 'quasar'

const generateNewSessionCol = [
  {
    name: 'select',
    label: '',
    align: 'center',
    style: 'width: 20%',
  },
  {
    name: 'category',
    align: 'left',
    label: 'Category',
    style: 'width: 20%',
  },
  {
    name: 'description',
    label: 'Description',
    align: 'left',
    style: 'width: 25%',
  },
  {
    name: 'version',
    label: 'version',
    align: 'left',
    style: 'width: 20%',
  },
  {
    name: 'status',
    label: 'status',
    align: 'left',
    style: 'width: 15%',
  },
]
const loadPreSessionCol = [
  {
    name: 'select',
    label: 'Session',
    align: 'center',
  },
  {
    name: 'zclproperty',
    label: 'ZCL Property',
    align: 'center',
  },
  {
    name: 'gen template file',
    align: 'left',
    label: 'Generation Template File',
  },
  {
    name: 'creation time',
    label: 'Creation Time',
    align: 'left',
  },
]

export default {
  name: 'ZapConfig',
  data() {
    return {
      customConfig: 'select',
      selected: [],
      selectedZclPropertiesData: null,
      selectedZclGenData: [],
      selectedZclSessionData: null,
      zclPropertiesRow: [],
      newSessionCol: generateNewSessionCol,
      loadPreSessionCol: loadPreSessionCol,
      zclGenRow: [],
      newConfig: false,
      path: window.location,
      open: true,
      filePath: '',
      loadPreSessionData: [],
      pagination: {
        rowsPerPage: 10,
      },
      newGenerationPagination: {
        rowsPerPage: 0,
      },
      propertyDataDialog: {},
      genDataDialog: {},
    }
  },
  computed: {
    disableSubmitButton: function () {
      if (this.customConfig === 'select')
        return (
          this.selectedZclPropertiesData == null ||
          this.selectedZclGenData.length == 0
        )
      else return this.selectedZclSessionData == null
    },
    isMultiplePackage: function () {
      return this.zclPropertiesRow.length > 1
    },
    getuitheme: function () {
      return this.selectedZclPropertiesData?.category
    },
    getLogo: {
      get() {
        if (this.selectedZclPropertiesData?.category) {
          return (
            '/' +
            this.selectedZclPropertiesData?.category +
            '_logo' +
            (this.$q.dark.isActive ? '_white' : '') +
            '.svg'
          )
        } else {
          return '/zap_logo.png'
        }
      },
    },
  },

  watch: {
    getuitheme() {
      this.addClassToBody()
    },
  },
  methods: {
    addClassToBody() {
      if (this.getuitheme === 'zigbee') {
        document.body.classList.remove('matter')
        document.body.classList.add('zigbee')
      } else {
        document.body.classList.remove('zigbee')
        document.body.classList.add('matter')
      }
    },
    submitForm() {
      if (this.customConfig === 'select') {
        let data = {
          zclProperties: this.selectedZclPropertiesData,
          genTemplate: this.selectedZclGenData,
        }
        this.$router.push({ path: '/' })
        this.$q.loading.show({
          spinner: QSpinnerGears,
          messageColor: 'white',
          message: 'Please wait while zap is loading...',
          spinnerSize: 300,
        })

        if (this.open) {
          this.$serverPost(restApi.uri.sessionCreate, data)
            .then(() => this.$serverPost(restApi.ide.open, this.path))
            .then(() => {
              this.$store.commit('zap/selectZapConfig', {
                zclProperties: this.selectedZclPropertiesData,
                genTemplate: this.selectedZclGenData,
                newConfig: false,
              })
            })
        } else {
          this.$serverPost(restApi.uri.sessionCreate, data).then(() => {
            this.$store.commit('zap/selectZapConfig', {
              zclProperties: this.selectedZclPropertiesData,
              genTemplate: this.selectedZclGenData,
              newConfig: true,
            })
          })
        }
      } else {
        this.$serverPost(restApi.uri.reloadSession, {
          sessionId: this.selectedZclSessionData.id,
        }).then((result) => {
          this.$store.commit('zap/selectZapConfig', {
            sessionId: this.selectedZclSessionData.id,
            zclProperties: this.selectedZclSessionData.zclProperty,
          })
        })
      }
    },
    // classify all notifications by type and package ref
    classifyMessages(data) {
      return data.reduce(
        (map, row) => {
          const key = row.type === 'ERROR' ? 'errorMap' : 'warningMap'
          if (!map[key][row.ref]) {
            map[key][row.ref] = []
          }
          map[key][row.ref].push(row.message)
          return map
        },
        { warningMap: {}, errorMap: {} }
      )
    },
    // then assign classfied notifications to proper packages
    assignWarningsAndErrors(data, warningMap, errorMap) {
      data.forEach((row) => {
        if (warningMap[row.id]) {
          row.hasWarning = true
          row.warnings = warningMap[row.id]
        } else {
          row.hasWarning = false
          row.warnings = []
        }
        if (errorMap[row.id]) {
          row.hasError = true
          row.errors = errorMap[row.id]
        } else {
          row.hasError = false
          row.errors = []
        }
        // set up var for dialog component of each package
        this.propertyDataDialog[row.id] = false
        this.genDataDialog[row.id] = false
      })
    },
  },
  created() {
    this.$serverPost(restApi.uri.sessionAttempt, this.path).then((result) => {
      this.zclPropertiesRow = result.data.zclProperties
      this.selectedZclPropertiesData = result.data.zclProperties[0]
      this.zclGenRow = result.data.zclGenTemplates
      this.filePath = result.data.filePath
      this.open = result.data.open
      if (this.zclPropertiesRow.length == 1) {
        // We shortcut this page, if there is exactly one of each,
        // since we simply assume that they are selected and move on.
        if (this.selectedZclGenData[0]) {
          this.selectedZclGenData[0] = this.zclGenRow[0].path
        }
        this.customConfig = 'select'
        this.submitForm()
      } else {
        this.customConfig = 'select'
      }

      result.data.sessions.forEach((item) => {
        let atts = null
        let gen = null
        item.packageRef.forEach((element) => {
          !atts
            ? (atts = this.zclPropertiesRow.find((data) => data.id === element))
            : ''
          !gen ? (gen = this.zclGenRow.find((data) => data.id === element)) : ''
        })
        this.loadPreSessionData.push({
          zclProperty: atts,
          genTemplateFile: gen,
          creationTime: item.creationTime,
          id: item.sessionId,
        })
      })

      // load package notification data, and assign to both zclProperty & zclGen packages
      this.$serverGet(restApi.uri.packageNotification)
        .then((resp) => {
          const { warningMap, errorMap } = this.classifyMessages(resp.data)
          this.assignWarningsAndErrors(
            this.zclPropertiesRow,
            warningMap,
            errorMap
          )
          this.assignWarningsAndErrors(this.zclGenRow, warningMap, errorMap)
        })
        .catch((err) => {
          console.log(err)
        })
    })
  },
}
</script>
<style lang="scss" scoped>
.q-page {
  background: url('/bg-matter.jpg');
  // background: @image, -moz-linear-gradient(top, @startColor, @endColor); // FF 3.6+
  //     background: @image, -webkit-gradient(linear, 0 0, 0 100%, from(@startColor), to(@endColor)); // Safari 4+, Chrome 2+
  //     background: @image, -webkit-linear-gradient(top, @startColor, @endColor); // Safari 5.1+, Chrome 10+
  //     background: @image, -o-linear-gradient(top, @startColor, @endColor); // Opera 11.10
  //     background: @image, linear-gradient(to bottom, @startColor, @endColor); // Standard, IE10
  //background-attachment: fixed;
  background-size: cover;
  transition: 2s;

  &.zigbee {
    background: url('/bg-zigbee.jpg');
    //  background-attachment: fixed;
    background-size: cover;
    transition: 2s;
  }
}
.slide-up-enter-active,
.slide-up-leave-active {
  transition: all 0.55s ease-out;
}

.slide-up-enter-from {
  opacity: 0;
  transform: translateY(30px);
}

.slide-up-leave-to {
  opacity: 0;
  transform: translateY(-30px);
}
</style>
