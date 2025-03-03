<template>
  <div class="text-center">
    <b-card
      v-if="calculateTotal > 0"
      border-variant="primary"
    >
      <b-row class="mx-0 d-flex align-items-center">
        <b-col
          md="4"
          class="py-0"
        >
          <b-dropdown
            :text="`Currency: ${currency2.toUpperCase()}`"
            size="sm"
            class="text-uppercase"
            variant="primary"
          >
            <b-dropdown-item @click="setCurrency('usd')">
              USD
            </b-dropdown-item>
            <b-dropdown-item @click="setCurrency('cny')">
              CNY (人民币)
            </b-dropdown-item>
            <b-dropdown-item @click="setCurrency('eur')">
              EUR (Euro)
            </b-dropdown-item>
            <b-dropdown-item @click="setCurrency('jpy')">
              JPY (日本円)
            </b-dropdown-item>
            <b-dropdown-item @click="setCurrency('hkd')">
              HKD (港幣)
            </b-dropdown-item>
            <b-dropdown-item @click="setCurrency('sgd')">
              SGD
            </b-dropdown-item>
            <b-dropdown-item @click="setCurrency('krw')">
              KRW (대한민국원)
            </b-dropdown-item>
          </b-dropdown>
          <h2 class="mt-1 mb-0">
            {{ currency }}{{ calculateTotal }}
          </h2>
          <small
            v-if="calculateTotalChange > 0"
            class="my-0 text-success"
          >
            +{{ calculateTotalChange }} (24h)
          </small>
          <small
            v-else
            class="my-0 text-danger"
          >
            {{ calculateTotalChange }} (24h)
          </small>
          <!-- chart -->
          <chart-component-doughnut
            :height="160"
            :width="160"
            :data="calculateChartDoughnut"
          />
        </b-col>
        <b-col
          md="8"
        >
          <chartjs-component-bar
            :height="200"
            :chart-data="calculateChartBar"
            :options="options"
          />
        </b-col>
      </b-row>
    </b-card>

    <div
      v-for="item,index in accounts"
      :key="index"
    >
      <div>
        <div class="d-flex justify-content-between align-items-end mb-1">
          <b-button
            v-ripple.400="'rgba(255, 255, 255, 0.15)'"
            variant="warning"
            :to="`/wallet/import?name=${item.name}`"
          >
            <feather-icon
              icon="EditIcon"
              class="mr-50"
            />
            <span class="align-middle">{{ item.name }}</span>
          </b-button>
          <b-form-checkbox
            v-model="defaultWallet"
            v-b-tooltip.hover.v-primary
            :value="item.name"
            title="Set as default wallet"
            variant="outline-warning"
            :readonly="item.name===defaultWallet"
          >
            <span
              :class="item.name===defaultWallet ? 'text-primary' : ''"
              class="font-weight-bolder pb-0"
              style="font-size:16px"
            >
              Set as default
            </span>
          </b-form-checkbox>
        </div>

        <b-row>
          <b-col
            v-for="acc, j in item.address"
            :key="j"
            sm="12"
            md="6"
            xl="4"
            :class="(balances[acc.addr])? 'order-1' : 'order-9' "
          >

            <b-card
              no-body
              class="card-browser-states"
            >
              <b-card-header>
                <div>
                  <b-card-title> <span class="text-uppercase">{{ acc.chain }}</span></b-card-title>
                </div>
                <b-dropdown
                  class="ml-1"
                  variant="link"
                  no-caret
                  toggle-class="p-0"
                  right
                >
                  <template #button-content>
                    <feather-icon
                      icon="AlignJustifyIcon"
                      size="18"
                      class="cursor-pointer"
                    />
                  </template>
                  <b-dropdown-item
                    v-if="balances[acc.addr]"
                    :to="`/${acc.chain}/account/${acc.addr}`"
                  >
                    <feather-icon icon="TrelloIcon" /> Detail
                  </b-dropdown-item>
                  <b-dropdown-divider
                    v-if="balances[acc.addr]"
                  />
                  <b-dropdown-item
                    v-if="balances[acc.addr]"
                    v-b-modal.transfer-window
                    @click="transfer(acc.addr)"
                  >
                    <feather-icon icon="SendIcon" /> Transfer
                  </b-dropdown-item>
                  <b-dropdown-item
                    v-if="balances[acc.addr]"
                    v-b-modal.ibc-transfer-window
                    @click="transfer(acc.addr)"
                  >
                    <feather-icon icon="SendIcon" /> IBC Transfer
                  </b-dropdown-item>
                  <b-dropdown-item @click="removeAddress(acc.addr)">
                    <feather-icon icon="Trash2Icon" /> Remove
                  </b-dropdown-item>
                </b-dropdown>

              </b-card-header>
              <b-card-body class="text-truncate">
                <b-row>
                  <b-col>
                    <div class="d-flex justify-content-between">
                      <b-avatar
                        :src="acc.logo"
                        size="28"
                        variant="light-primary"
                        rounded
                      />
                      <div class="text-right">
                        <h4 class="my-0">{{ currency }}{{ formatBalance(acc.addr) }}
                        </h4>
                        <small :class="formatBalanceChangesColor(acc.addr)"> {{ formatBalanceChanges(acc.addr) }}</small>
                      </div>
                    </div>
                    <app-collapse>
                      <app-collapse-item title="Assets">
                        <template #header>
                          <div>
                            <feather-icon
                              icon="CopyIcon"
                              @click="copy(acc.addr)"
                            />&nbsp;
                            <small class="text-muted">{{ formatAddr(acc.addr) }}</small>
                          </div>
                        </template>
                        <div
                          v-for="b,i in balances[acc.addr]"
                          :key="i"
                          class="d-flex justify-content-between"
                        >
                          <div
                            class="ml-25 font-weight-bolder text-uppercase text-success d-flex flex-column text-left"
                            title="Balance"
                          >
                            {{ formatAmount(b.amount, b.denom) }} {{ formatDenom(b.denom) }}
                            <span class="font-small-2 text-muted text-nowrap">{{ currency }}{{ formatCurrency(b.amount, b.denom) }}</span>
                          </div>
                          <div class="d-flex flex-column text-right">
                            <span class="font-weight-bold mb-0">{{ currency }}{{ formatPrice(b.denom) }}</span>
                            <small
                              :class="priceColor(b.denom)"
                              class="py-0"
                            >{{ formatChanges(b.denom) }}</small>
                          </div>
                        </div>
                        <div
                          v-for="b,i in delegations[acc.addr]"
                          :key="`d-${i}`"
                          class="d-flex justify-content-between align-items-center"
                        >
                          <div
                            class="ml-25 font-weight-bolder text-uppercase text-primary d-flex flex-column text-left"
                            title="Balance"
                          >
                            {{ formatAmount(b.amount, b.denom) }} {{ formatDenom(b.denom) }}
                            <span class="font-small-2 text-muted text-nowrap">{{ currency }}{{ formatCurrency(b.amount, b.denom) }}</span>
                          </div>
                          <div class="d-flex flex-column text-right">
                            <span class="font-weight-bold mb-0">{{ currency }}{{ formatPrice(b.denom) }}</span>
                            <small
                              :class="priceColor(b.denom)"
                              class="py-0"
                            >{{ formatChanges(b.denom) }}</small>
                          </div>
                        </div>
                        <b-button
                          v-if="balances[acc.addr]"
                          block
                          size="sm"
                          variant="outline-primary"
                          :to="`/${acc.chain}/account/${acc.addr}`"
                          class="mt-1 mb-0"
                        >
                          <feather-icon icon="TrelloIcon" /> Detail
                        </b-button>
                      </app-collapse-item>
                    </app-collapse>
                  </b-col>
                </b-row>
              </b-card-body>
            </b-card>
          </b-col>
        </b-row>
      </div>
    </div>

    <router-link to="/wallet/import">
      <b-card class="addzone">
        <feather-icon icon="PlusIcon" />
        Connect Wallet
      </b-card>
    </router-link>
    <operation-transfer-component
      :address.sync="selectedAddress"
    />
    <operation-transfer-2-component :address="selectedAddress" />
  </div>
</template>

<script>
import {
  BCard, BCardHeader, BCardTitle, BCardBody, VBModal, BRow, BCol, BAvatar, BButton,
  BDropdown, BDropdownItem, BDropdownDivider, BFormCheckbox, VBTooltip,
} from 'bootstrap-vue'
import Ripple from 'vue-ripple-directive'
import FeatherIcon from '@/@core/components/feather-icon/FeatherIcon.vue'
import {
  chartColors,
  formatNumber,
  formatTokenAmount, formatTokenDenom, getLocalAccounts, getLocalChains, getUserCurrency, getUserCurrencySign, setUserCurrency,
} from '@/libs/utils'
import ToastificationContent from '@core/components/toastification/ToastificationContent.vue'
import AppCollapse from '@core/components/app-collapse/AppCollapse.vue'
import AppCollapseItem from '@core/components/app-collapse/AppCollapseItem.vue'
import OperationTransferComponent from './OperationTransferComponent.vue'
import OperationTransfer2Component from './OperationTransfer2Component.vue'
import ChartComponentDoughnut from './ChartComponentDoughnut.vue'
import ChartjsComponentBar from './ChartjsComponentBar.vue'

export default {
  components: {
    BAvatar,
    BCard,
    BRow,
    BButton,
    BCol,
    BCardHeader,
    BCardBody,
    BCardTitle,
    BDropdown,
    BDropdownItem,
    BDropdownDivider,
    BFormCheckbox,
    // eslint-disable-next-line vue/no-unused-components
    VBTooltip,
    FeatherIcon,
    OperationTransferComponent,
    // eslint-disable-next-line vue/no-unused-components
    ToastificationContent,
    OperationTransfer2Component,
    ChartComponentDoughnut,
    ChartjsComponentBar,
    AppCollapse,
    AppCollapseItem,
  },
  directives: {
    'b-tooltip': VBTooltip,
    'b-modal': VBModal,
    Ripple,
  },
  data() {
    return {
      currency: getUserCurrencySign(),
      currency2: getUserCurrency(),
      selectedAddress: '',
      selectedName: '',
      transferWindow: false,
      accounts: [],
      balances: {},
      delegations: {},
      ibcDenom: {},
      quotes: {},
      options: {
        maintainAspectRatio: false,
        legend: {
          display: false,
        },
        // responsive: true,
        title: {
          display: true,
          text: 'Token Portfolio',
        },
        tooltips: {
          mode: 'index',
          intersect: true,
        },
        scales: {
          yAxes: [{
            // type: 'linear', // only linear but allow scale type registration. This allows extensions to exist solely for log scale for instance
            type: 'logarithmic',
            display: true,
            position: 'left',
            id: 'y-axis-1',
            ticks: {
              min: 0,
              autoskipping: true,
              display: true,
              maxTicksLimit: 7,
              callback(val) {
                return formatNumber(val, true, 0)
              },
              color: 'red',
            },
            offset: true,
            gridLines: {
              display: true,
              offsetGridLines: false,
            },
          }, {
            type: 'linear', // only linear but allow scale type registration. This allows extensions to exist solely for log scale for instance
            display: false,
            position: 'right',
            id: 'y-axis-2',
            gridLines: {
              drawOnChartArea: true,
            },
          }],
        },
      },
    }
  },
  computed: {
    defaultWallet: {
      get() {
        return this.$store.state.chains.defaultWallet
      },
      set(value) {
        this.$store.commit('setDefaultWallet', value)
      },
    },
    calculateTotal() {
      const v = Object.values(this.balances)
      let total = 0
      if (v) {
        v.forEach(tokens => {
          const subtotal = tokens.map(x => this.formatCurrency(x.amount, x.denom)).reduce((t, c) => t + c)
          total += subtotal
        })
      }
      const d = Object.values(this.delegations)
      if (d) {
        d.forEach(tokens => {
          const subtotal = tokens.map(x => this.formatCurrency(x.amount, x.denom)).reduce((t, c) => t + c, 0)
          total += subtotal
        })
      }
      return parseFloat(total.toFixed(2))
    },
    calculateTotalChange() {
      const v = Object.values(this.balances)
      let total = 0
      if (v) {
        v.forEach(tokens => {
          const subtotal = tokens.map(x => this.formatCurrency(x.amount, x.denom) * this.getChanges(x.denom) * 0.01).reduce((t, c) => t + c)
          total += subtotal
        })
      }
      const d = Object.values(this.delegations)
      if (d) {
        d.forEach(tokens => {
          const subtotal = tokens.map(x => this.formatCurrency(x.amount, x.denom) * this.getChanges(x.denom) * 0.01).reduce((t, c) => t + c, 0)
          total += subtotal
        })
      }
      return parseFloat(total.toFixed(2))
    },
    calculateByDenom() {
      const v = Object.values(this.balances)
      const total = {}
      const qty = {}
      if (v) {
        v.forEach(tokens => {
          tokens.forEach(x => {
            const denom = this.formatDenom(x.denom)
            if (total[denom]) {
              total[denom] += this.formatCurrency(x.amount, x.denom)
            } else {
              total[denom] = this.formatCurrency(x.amount, x.denom)
            }
            if (qty[denom]) {
              qty[denom] += this.formatAmount(x.amount, x.denom)
            } else {
              qty[denom] = this.formatAmount(x.amount, x.denom)
            }
          })
        })
      }
      const d = Object.values(this.delegations)
      if (d) {
        d.forEach(tokens => {
          tokens.forEach(x => {
            const denom = this.formatDenom(x.denom)
            if (total[denom]) {
              total[denom] += this.formatCurrency(x.amount, x.denom)
            } else {
              total[denom] = this.formatCurrency(x.amount, x.denom)
            }
            if (qty[denom]) {
              qty[denom] += this.formatAmount(x.amount, x.denom)
            } else {
              qty[denom] = this.formatAmount(x.amount, x.denom)
            }
          })
        })
      }
      return { value: total, qty }
    },
    calculateChartDoughnut() {
      const total = this.calculateByDenom
      return {
        datasets: [
          {
            labels: Object.keys(total.value),
            data: Object.values(total.value),
            backgroundColor: chartColors(),
            borderWidth: 0,
            pointStyle: 'rectRounded',
          },
        ],
      }
    },
    calculateChartBar() {
      const total = this.calculateByDenom
      // Object.entries(total.value).sort((a, b) => b[0].localeCompare(a[0]));
      const data = Object.entries(total.value).sort((a, b) => b[1] - a[1]).slice(0, 15)
      return {
        labels: data.map(x => x[0]),
        datasets: [
          {
            label: 'Market Cap',
            data: data.map(x => x[1]),
            backgroundColor: chartColors(),
            borderWidth: 0,
            pointStyle: 'rectRounded',
            yAxisID: 'y-axis-1',
          },
          {
            label: 'Qty',
            data: data.map(x => total.qty[x[0]]), // Object.values(total.qty),
            backgroundColor: chartColors(),
            borderWidth: 0,
            pointStyle: 'rectRounded',
            yAxisID: 'y-axis-2',
          },
        ],
      }
    },
  },
  created() {
    this.init()
  },
  mounted() {
  },
  methods: {
    init() {
      this.accounts = getLocalAccounts()
      const chains = getLocalChains()
      if (this.accounts) {
        Object.keys(this.accounts).forEach(acc => {
          this.accounts[acc].address.forEach(add => {
            this.$http.getBankBalances(add.addr, chains[add.chain]).then(res => {
              if (res && res.length > 0) {
                this.$set(this.balances, add.addr, res)
                res.forEach(token => {
                  if (token.denom.startsWith('ibc')) {
                    this.$http.getIBCDenomTrace(token.denom, chains[add.chain]).then(denom => {
                      this.$set(this.ibcDenom, token.denom, denom)
                    })
                  }
                })
              }
            })
            this.$http.getStakingDelegations(add.addr, chains[add.chain]).then(res => {
              if (res.delegation_responses) {
                const delegation = res.delegation_responses.map(x => x.balance).reduce((t, c) => {
                  const t1 = t
                  if (t1[c.denom]) {
                    t1[c.denom] += Number(c.amount)
                  } else {
                    t1[c.denom] = Number(c.amount)
                  }
                  return t1
                }, {})
                this.$set(this.delegations, add.addr, Object.keys(delegation).map(x => ({ amount: String(delegation[x]), denom: x })))
              }
            }).catch(() => {})
          })
        })
      }
    },
    setCurrency(c) {
      setUserCurrency(c)
      this.currency2 = c
      this.currency = getUserCurrencySign()
    },
    transfer(addr) {
      this.selectedAddress = addr
    },
    completeAdd() {
      this.init()
      this.$bvModal.hide('add-account')
    },
    formatDenom(v) {
      if (!v) return ''
      const denom = (v.startsWith('ibc') ? this.ibcDenom[v] : v)
      return formatTokenDenom(denom)
    },
    formatAmount(v, denom = 'uatom') {
      if (!v) return ''
      const denom2 = (denom.startsWith('ibc') ? this.ibcDenom[denom] : denom)
      return formatTokenAmount(v, 2, denom2)
    },
    formatAddr(v) {
      return v.substring(0, 10).concat('...', v.substring(v.length - 10))
    },
    formatCurrency(amount, denom) {
      const qty = this.formatAmount(amount, denom)
      const d2 = this.formatDenom(denom)
      const quote = this.$store.state.chains.quotes[d2]
      if (quote) {
        const price = quote[this.currency2]
        return parseFloat((qty * price).toFixed(2))
      }
      return 0
    },
    priceColor(denom) {
      const d2 = this.formatDenom(denom)
      const quote = this.$store.state.chains.quotes[d2]
      if (quote) {
        const price = quote[`${this.currency2}_24h_change`]
        return price > 0 ? 'text-success' : 'text-danger'
      }
      return ''
    },
    getChanges(denom) {
      const d2 = this.formatDenom(denom)
      const quote = this.$store.state.chains.quotes[d2]
      if (quote) {
        const price = quote[`${this.currency2}_24h_change`]
        return price
      }
      return 0
    },
    formatChanges(denom) {
      const price = this.getChanges(denom)
      return price === 0 ? '' : `${parseFloat(price.toFixed(2))}%`
    },
    formatPrice(denom) {
      const d2 = this.formatDenom(denom)
      const quote = this.$store.state.chains.quotes[d2]
      if (quote) {
        const price = quote[this.currency2]
        return price
      }
      return 0
    },
    formatBalance(v) {
      let total = 0
      const balance = this.balances[v]
      if (balance) {
        const ret = balance.map(x => this.formatCurrency(x.amount, x.denom)).reduce((t, c) => t + c)
        total += ret
      }
      const delegations = this.delegations[v]
      if (delegations) {
        const ret = delegations.map(x => this.formatCurrency(x.amount, x.denom)).reduce((t, c) => t + c, 0)
        total += ret
      }
      return parseFloat(total.toFixed(2))
    },
    formatBalanceChanges(v) {
      let total = 0
      const balance = this.balances[v]
      if (balance) {
        const ret = balance.map(x => this.formatCurrency(x.amount, x.denom) * this.getChanges(x.denom) * 0.01).reduce((t, c) => t + c)
        total += ret
      }
      const delegations = this.delegations[v]
      if (delegations) {
        const ret = delegations.map(x => this.formatCurrency(x.amount, x.denom) * this.getChanges(x.denom) * 0.01).reduce((t, c) => t + c, 0)
        total += ret
      }
      return parseFloat(total.toFixed(2))
    },
    formatBalanceChangesColor(v) {
      const total = this.formatBalanceChanges(v)
      return total > 0 ? 'text-success' : 'text-danger'
    },
    removeAddress(v) {
      Object.keys(this.accounts).forEach(key => {
        const item = this.accounts[key]
        const newAddrs = item.address.filter(a => a.addr !== v)
        if (newAddrs.length > 0) {
          this.$set(item, 'address', newAddrs)
        } else {
          delete this.accounts[key]
        }
      })
      localStorage.setItem('accounts', JSON.stringify(this.accounts))
    },
    copy(v) {
      this.$copyText(v).then(() => {
        this.$toast({
          component: ToastificationContent,
          props: {
            title: 'Address copied',
            icon: 'BellIcon',
          },
        })
      }, e => {
        this.$toast({
          component: ToastificationContent,
          props: {
            title: `Failed to copy address! ${e}`,
            icon: 'BellIcon',
            variant: 'danger',
          },
        })
      })
    },
  },
}
</script>

<style lang="css">
.addzone {
    border: 2px dashed #ced4da;
    background: #fff;
    border-radius: 6px;
    cursor: pointer;
    box-shadow: none;
}
.addzone :hover {
    border: 2px dashed #362c63;
}

</style>
