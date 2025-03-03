<template>
  <section>
    <b-card
      no-body
    >
      <b-card-header>
        <b-card-title>
          #{{ proposal.id }}
          <b-badge
            v-if="proposal.status == 1"
            pill
            variant="light-info"
            class="text-right"
          >
            Deposit
          </b-badge>
          <b-badge
            v-if="proposal.status == 2"
            pill
            variant="light-primary"
            class="text-right"
          >
            Voting
          </b-badge>
          <b-badge
            v-if="proposal.status == 3"
            pill
            variant="light-success"
            class="text-right"
          >
            Passed
          </b-badge>
          <b-badge
            v-if="proposal.status == 4"
            pill
            variant="light-danger"
            class="text-right"
          >
            Rejected
          </b-badge>
          {{ proposal.title }}
        </b-card-title>
      </b-card-header>
      <b-card-body>
        <b-table-simple
          stacked="sm"
          hover
          striped
        >
          <tbody>
            <b-tr>
              <b-td
                style="text-transform: capitalize; vertical-align: top; width:200px"
              >
                {{ $t('proposal_id') }}
              </b-td><b-td>{{ proposal.id }}</b-td>
            </b-tr>
            <b-tr>
              <b-td>
                {{ $t('proposal_proposer') }}
              </b-td><b-td><router-link :to="`../account/${proposer.proposer}`">
                {{ formatAddress(proposer.proposer) }}
              </router-link> </b-td>
            </b-tr>
            <b-tr>
              <b-td>
                {{ $t('proposal_total_deposit') }}
              </b-td><b-td>{{ formatToken(proposal.total_deposit) }} </b-td>
            </b-tr>
            <b-tr>
              <b-td>
                {{ $t('proposal_submit_time') }}
              </b-td><b-td>{{ formatDate(proposal.submit_time) }}</b-td>
            </b-tr>
            <b-tr>
              <b-td>
                {{ $t('voting_time') }}
              </b-td><b-td>{{ formatDate(proposal.voting_start_time) }} - {{ formatDate(proposal.voting_end_time) }}</b-td>
            </b-tr>
            <b-tr>
              <b-td>
                {{ $t('proposal_type') }}
              </b-td><b-td>
                {{ proposal.type }}
              </b-td>
            </b-tr>
          </tbody>
        </b-table-simple>
        <div style="white-space: pre-line">
          <object-field-component
            :tablefield="proposal.contents"
            :small="false"
          /></div>
        <b-table-simple v-if="proposal.type === 'cosmos-sdk/SoftwareUpgradeProposal'">
          <b-tr>
            <b-td class="text-center">
              {{ $t('upgrade_time') }} {{ upgradeTime }}
              <flip-countdown :deadline="upgradeTime" />
            </b-td>
          </b-tr>
        </b-table-simple>
      </b-card-body>
      <b-card-footer>
        <router-link :to="`../gov`">
          <b-button
            variant="outline-primary"
          >
            {{ $t('btn_back_list') }}
          </b-button>
        </router-link>
        <b-button
          v-b-modal.vote-window
          :disabled="proposal.status!=2"
          variant="primary"
          class="btn float-right mg-2"
        >
          {{ $t('btn_vote') }}
        </b-button>
      </b-card-footer>
    </b-card>
    <b-card no-body>
      <b-card-header>
        <b-card-title>
          Votes
        </b-card-title>
      </b-card-header>
      <b-card-body>
        <b-progress
          :max="100"
          height="2rem"
          class="mb-2"
          show-progress
        >
          <b-progress-bar
            :id="'vote-yes'+proposal.id"
            variant="success"
            :value="percent(proposal.tally.yes)"
            :label="`${percent(proposal.tally.yes).toFixed()}%`"
            show-progress
          />
          <b-progress-bar
            :id="'vote-no'+proposal.id"
            variant="warning"
            :value="percent(proposal.tally.no)"
            :label="`${percent(proposal.tally.no).toFixed()}%`"
            show-progress
          />
          <b-progress-bar
            :id="'vote-veto'+proposal.id"
            variant="danger"
            :value="percent(proposal.tally.veto)"
            :label="`${percent(proposal.tally.veto).toFixed()}%`"
            show-progress
          />
          <b-progress-bar
            :id="'vote-abstain'+proposal.id"
            variant="info"
            :value="percent(proposal.tally.abstain)"
            :label="`${percent(proposal.tally.abstain).toFixed()}%`"
            show-progress
          />
        </b-progress>
        <b-tooltip
          :target="'vote-yes'+proposal.id"
        >
          {{ percent(proposal.tally.yes) }}% voted Yes
        </b-tooltip>
        <b-tooltip
          :target="'vote-no'+proposal.id"
        >
          {{ percent(proposal.tally.no) }}% voted No
        </b-tooltip>
        <b-tooltip
          :target="'vote-veto'+proposal.id"
        >
          {{ percent(proposal.tally.veto) }}% voted No With Veto
        </b-tooltip>
        <b-tooltip
          :target="'vote-abstain'+proposal.id"
        >
          {{ percent(proposal.tally.abstain) }}% voted Abstain
        </b-tooltip>
        <b-table
          v-if="votes.votes && votes.votes.length > 0"
          stacked="sm"
          :fields="votes_fields"
          :items="votes.votes"
          striped
        >
          <template #cell(voter)="data">
            <router-link :to="`../account/${data.item.voter}`">
              {{ formatAddress(data.item.voter) }}
            </router-link>
          </template>
        </b-table>
        <div
          v-if="next"
          class="addzone text-center pt-50 pb-50 bg-transparent text-primary"
          @click="loadVotes()"
        >
          <feather-icon icon="PlusIcon" />
          Load More Votes
        </div>
      </b-card-body>
    </b-card>
    <b-card no-body>
      <b-card-header>
        <b-card-title>
          Deposits ({{ formatToken(proposal.total_deposit) }})
        </b-card-title>
      </b-card-header>
      <b-card-body>
        <b-table
          stacked="sm"
          :items="deposits.deposits?deposits.deposits:deposits"
          :fields="deposit_fields"
          striped
        >
          <template #cell(depositor)="data">
            <router-link :to="`../account/${data.item.depositor}`">
              {{ formatAddress(data.item.depositor) }}
            </router-link>
          </template>
        </b-table>
      </b-card-body>
      <b-card-footer>
        <router-link :to="`../gov`">
          <b-button
            variant="outline-primary"
          >
            {{ $t('btn_back_list') }}
          </b-button>
        </router-link>
        <b-button
          v-b-modal.deposit-window
          :disabled="proposal.status!=1"
          variant="primary"
          class="btn float-right mg-2"
        >
          {{ $t('btn_deposit') }}
        </b-button>
        <b-button
          v-b-modal.vote-window
          :disabled="proposal.status!=2"
          variant="primary"
          class="btn float-right mg-2 mr-1"
        >
          {{ $t('btn_vote') }}
        </b-button>
      </b-card-footer>
    </b-card>
    <operation-vote-component
      :proposal-id="Number(proposal.id)"
      :title="proposal.title"
    />
    <operation-gov-deposit-component
      :proposal-id="Number(proposal.id)"
      :title="proposal.title"
    />
  </section>
</template>

<script>
import {
  BCard, BCardBody, BCardFooter, BButton, BTable, BTableSimple, BTr, BTd, BCardTitle, BCardHeader,
  BProgressBar, BProgress, BTooltip, BBadge,
} from 'bootstrap-vue'
import FlipCountdown from 'vue2-flip-countdown'
// import fetch from 'node-fetch'

import {
  getCachedValidators, getStakingValidatorByAccount, percent, tokenFormatter,
} from '@/libs/utils'
import { Proposal, Proposer } from '@/libs/data'
import dayjs from 'dayjs'
import ObjectFieldComponent from './ObjectFieldComponent.vue'
import OperationVoteComponent from './OperationVoteComponent.vue'
import OperationGovDepositComponent from './OperationGovDepositComponent.vue'
// import { formatToken } from '@/libs/data/data'

export default {
  components: {
    BCard,
    BCardTitle,
    BCardBody,
    BCardFooter,
    BButton,
    BTable,
    BTableSimple,
    BCardHeader,
    BTr,
    BTd,
    BProgressBar,
    BProgress,
    BTooltip,
    BBadge,
    ObjectFieldComponent,
    OperationVoteComponent,
    OperationGovDepositComponent,
    FlipCountdown,
  },
  data() {
    return {
      latest: {},
      next: null,
      proposal: new Proposal(),
      proposer: new Proposer(),
      deposits: [],
      votes: [],
      votes_fields: [
        {
          key: 'voter',
          sortable: true,
          formatter: v => this.formatAddress(v),
        },
        {
          key: 'option',
          sortable: true,
          formatter: value => {
            switch (value) {
              case 1:
              case 'VOTE_OPTION_YES':
                return 'Yes'
              case 2:
              case 'VOTE_OPTION_ABSTAIN':
                return 'Abstain'
              case 3:
              case 'VOTE_OPTION_NO':
                return 'No'
              case 4:
              case 'VOTE_OPTION_NO_WITH_VETO':
                return 'No With Veto'
              default:
                return value
            }
          },
        },
      ],
      deposit_fields: [
        {
          key: 'depositor',
          formatter: v => this.formatAddress(v),
        },
        {
          key: 'amount',
          sortable: true,
          formatter: tokenFormatter,
        },
      ],
    }
  },
  computed: {
    upgradeTime() {
      if (this.proposal.type === 'cosmos-sdk/SoftwareUpgradeProposal') {
        if (Number(this.proposal?.contents.plan.height || 0) > 0 && this.latest?.block) {
          const blocks = Number(this.proposal.contents.plan.height) - Number(this.latest.block?.header?.height || 0)
          if (blocks > 0) {
            const endtime = dayjs().add(blocks * 6, 'second').format('YYYY-MM-DD HH:mm:ss')
            return endtime
          }
        }
        return dayjs(this.proposal.contents.plan.time).format('YYYY-MM-DD HH:mm:ss')
      }
      return '0001-01-01 00:00:00'
    },
  },
  created() {
    const pid = this.$route.params.proposalid

    this.$http.getLatestBlock().then(res => {
      this.latest = res
    })

    this.$http.getGovernance(pid).then(p => {
      if (p.status === 2) {
        this.$http.getGovernanceTally(pid, 0).then(t => p.updateTally(t))
      }
      this.proposal = p
    })

    if (!getCachedValidators()) {
      this.$http.getValidatorList()
    }

    this.$http.getGovernanceProposer(pid).then(res => {
      this.proposer = res
    })
    this.$http.getGovernanceDeposits(pid).then(res => {
      this.deposits = res
    })
    this.$http.getGovernanceVotes(pid).then(res => {
      this.votes = res
      this.next = res.pagination ? res.pagination.next_key : null
    })
  },
  methods: {
    percent: v => percent(v),
    formatDate: v => dayjs(v).format('YYYY-MM-DD HH:mm'),
    formatToken: v => tokenFormatter(v, {}),
    loadVotes() {
      if (this.next) {
        const pid = this.$route.params.proposalid
        const { next } = this
        this.next = null
        this.$http.getGovernanceVotes(pid, next).then(res => {
          this.$set(this.votes, 'votes', this.votes.votes.concat(res.votes))
          this.next = res.pagination ? res.pagination.next_key : null
        })
      }
    },
    formatAddress(v) {
      return getStakingValidatorByAccount(this.$http.config.chain_name, v)
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
@media (min-width: 768px) {
  td:first-child { width: 20% ;}
}
</style>
