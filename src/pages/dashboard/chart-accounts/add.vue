<template>
<div class="q-ma-md">
    <generic-form :parent-account="parentAccount" :chart-account="chartAccount" :selected-chart-account="selectedChartAccount" :show="true">
        <div class="col-12">
            <q-btn @click="cancel" color="secondary" label="Back" class="q-ma-sm" />
            <q-btn @click="add" color="primary" label="Submit" class="q-ma-sm" />
        </div>
    </generic-form>
</div>
</template>

<script>
import genericForm from 'pages/dashboard/chart-accounts/form/generic-form'
import {
  required,
  numeric
} from 'vuelidate/lib/validators'
import {
  mapState,
  mapActions
} from 'vuex'
export default {
  components: {
    genericForm
  },
  validations: {
    chartAccount: {
      name: {
        required

      },
      account_code: {
        required,
        numeric
      },
      account_display: {
        required

      },
      taccount_id: {
        required
      }
    }
  },
  computed: {
    ...mapState('chartAccounts', ['chartAccount', 'selectedChartAccount', 'parentAccount']),
    ...mapState('global', ['page', 'perPage', 'filter']),
    tAccounts () {
      return this.$store.getters['tAccounts/tAccounts'].map(e => {
        return {
          label: e.name,
          value: e.id
        }
      })
    }

  },
  methods: {
    ...mapActions('chartAccounts', ['setChartAccount', 'setParentAccount', 'setChartAccounts']),
    ...mapActions('tAccounts', ['setTAccounts']),
    getChartAccounts () {
      this.$axios.get(`/chart_accounts?page=${this.page}&perPage=${this.perPage}&filter=${this.filter}`)
        .then(res => {
          this.setChartAccounts(res.data.chartAccounts)
        })
    },
    cancel () {
      this.$router.go(-1)
    },
    add () {
      this.$v.$touch()
      if (this.$v.$invalid) {
        this.$q.notify({
          color: 'negative',
          icon: 'warning',
          message: `Please check the form fields.`
        })
      } else {
        this.$axios.post(`chart_accounts`, {
          taccount_id: this.chartAccount.taccount_id.value,
          parent_code: this.selectedChartAccount,
          name: this.chartAccount.name,
          account_code: this.chartAccount.account_code,
          account_display: this.chartAccount.account_display
        }).then(res => {
          this.$q.notify({
            color: 'positive',
            icon: 'check',
            message: `${this.chartAccount.name} created successfully.`
          })
          this.getChartAccounts()
          this.setChartAccount({
            name: '',
            account_code: '',
            account_display: '',
            remarks: '',
            taccount_id: ''
          })
        })
      }
    },
    getParentChartAccount () {
      this.$axios.get(`chart_accounts/create?account_code=${this.selectedChartAccount}`)
        .then(res => {
          this.setParentAccount(res.data.parentAccount)
          this.setTAccounts(res.data.tAccounts)
        })
    }
  },
  mounted () {
    this.getParentChartAccount()
  },
  watch: {
    chartAccount: {
      handler (val) {
        this.setChartAccount(val)
      },
      deep: true
    }
  }
}
</script>
