<script>
import FooterComponent from '@/components/form/Footer';
import SelectPrincipal from '@/components/auth/SelectPrincipal.vue';
import GlobalRoleBindings from '@/components/GlobalRoleBindings.vue';
import { NORMAN } from '@/config/types';
import { _VIEW } from '@/config/query-params';
import { exceptionToErrorsArray } from '@/utils/error';
import { NAME } from '@/config/product/auth';

export default {
  components: {
    SelectPrincipal,
    FooterComponent,
    GlobalRoleBindings,
  },
  data() {
    return {
      errors:       [],
      principalId:  null,
    };
  },
  computed: {
    mode() {
      return !this.principalId ? _VIEW : this.$route.query.mode || _VIEW;
    }
  },
  methods: {
    setPrincipal(id) {
      this.principalId = id;

      return true;
    },
    async cancel() {
      await this.return();
    },
    async save(buttonDone) {
      this.errors = [];

      try {
        await this.$refs.grb.save();

        await this.$store.dispatch('cluster/findAll', {
          type: NORMAN.SPOOFED.GROUP_PRINCIPAL,
          opt:  { force: true }
        }, { root: true }); // See PromptRemove.vue

        this.$router.replace({
          name:   `c-cluster-product-resource`,
          params: {
            cluster:  'local',
            product:  NAME,
            resource: NORMAN.SPOOFED.GROUP_PRINCIPAL,
          },
        });

        buttonDone(true);
      } catch (err) {
        this.errors = exceptionToErrorsArray(err);
        buttonDone(false);
      }
    },
  }
};

</script>

<template>
  <div>
    <div>
      <div class="masthead">
        <header>
          <div class="title">
            <h1 class="m-0">
              {{ t('authGroups.assignEdit.assignTitle') }}
            </h1>
          </div>
        </header>
      </div>

      <form>
        <SelectPrincipal :retain-selection="true" class="mb-20" :show-my-group-types="['group']" :search-group-types="'group'" @add="setPrincipal" />

        <GlobalRoleBindings ref="grb" :principal-id="principalId" :mode="mode" />

        <FooterComponent
          :mode="mode"
          :errors="errors"
          @save="save"
          @done="cancel"
        >
        </footercomponent>
      </form>
    </div>
  </div>
</template>

<style lang="scss" scoped>
  .masthead {
    padding-bottom: 10px;
    border-bottom: 1px solid var(--border);
    margin-bottom: 10px;
  }
  HEADER {
    margin: 0;
  }
  .actions {
    display: flex;
    justify-content: flex-end;
    align-items:center;

    .btn {
      margin-left: 10px;
    }
  }
</style>
