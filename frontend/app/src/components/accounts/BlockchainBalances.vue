<template>
  <v-card class="blockchain-balances mt-8">
    <v-card-title>Blockchain Balances</v-card-title>
    <v-card-text>
      <v-btn absolute fab top right dark color="primary" @click="newAccount()">
        <v-icon>
          mdi-plus
        </v-icon>
      </v-btn>
      <big-dialog
        :display="openDialog"
        :title="dialogTitle"
        :subtitle="dialogSubtitle"
        primary-action="Save"
        @confirm="save()"
        @cancel="clearDialog()"
      >
        <account-form ref="dialogChild" :edit="accountToEdit" />
      </big-dialog>
      <asset-balances
        title="Blockchain Balances per Asset"
        :balances="totals"
      />
      <v-divider />
      <account-balances
        title="ETH per account balances"
        blockchain="ETH"
        :balances="ethAccounts"
        @editAccount="edit($event)"
      />
      <v-divider />
      <account-balances
        title="BTC per account balances"
        blockchain="BTC"
        :balances="btcAccounts"
        @editAccount="edit($event)"
      />
    </v-card-text>
  </v-card>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { createNamespacedHelpers } from 'vuex';
import AccountBalances from '@/components/accounts/AccountBalances.vue';
import AccountForm from '@/components/accounts/AccountForm.vue';
import BigDialog from '@/components/dialogs/BigDialog.vue';
import AssetBalances from '@/components/settings/AssetBalances.vue';
import { AccountBalance } from '@/model/blockchain-balances';
import { Account } from '@/typing/types';

const { mapGetters } = createNamespacedHelpers('balances');

@Component({
  components: {
    AccountForm,
    AccountBalances,
    AssetBalances,
    BigDialog
  },
  computed: {
    ...mapGetters(['ethAccounts', 'btcAccounts', 'totals'])
  }
})
export default class BlockchainBalances extends Vue {
  ethAccounts!: AccountBalance[];
  btcAccounts!: AccountBalance[];
  totals!: AccountBalance[];

  accountToEdit: Account | null = null;
  dialogTitle: string = '';
  dialogSubtitle: string = '';
  openDialog: boolean = false;

  newAccount() {
    this.accountToEdit = null;
    this.dialogTitle = 'Add Blockchain Account';
    this.dialogSubtitle = '';
    this.openDialog = true;
  }
  edit(account: Account) {
    this.accountToEdit = account;
    this.dialogTitle = 'Edit Blockchain Account';
    this.dialogSubtitle = 'Modify account details';
    this.openDialog = true;
  }

  async clearDialog() {
    interface dataForm extends Vue {
      editComplete(): void;
    }
    const form = this.$refs.dialogChild as dataForm;
    form.editComplete();
    this.openDialog = false;
  }

  async save() {
    interface dataForm extends Vue {
      addAccount(): Promise<boolean>;
    }
    const form = this.$refs.dialogChild as dataForm;
    form.addAccount().then(success => {
      if (success === true) this.clearDialog();
    });
  }
}
</script>
