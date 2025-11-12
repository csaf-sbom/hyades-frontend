<template>
  <b-card no-body :header="header">
    <b-card-body>
      <c-switch
        color="primary"
        id="vulnsourceEnabled"
        label
        v-bind="labelIcon"
        v-model="vulnsourceEnabled"
        :disabled="!configInitialized"
      />
      {{ $t('admin.vulnsource_csaf_advisories_enable') }}
      <hr />
      <div>
        <b-card no-body>
          <b-tabs pills card>
            <b-tab title="CSAF Sources" active>
              <!--<b-card no-body :header="header">-->
              <b-card-body>
                <div id="repositoryToolbar" class="bs-table-custom-toolbar">
                  <b-button
                    size="md"
                    variant="outline-primary"
                    @click="triggerAll"
                    :disabled="!vulnsourceEnabled || triggerAllDisabled"
                    v-b-tooltip.hover
                    :title="triggerAllTooltip"
                  >
                    <span class="fa fa-refresh"></span>
                    {{ $t('admin.trigger_all') }}
                  </b-button>
                </div>
                <hr
                  style="
                    border: none;
                    height: 1px;
                    background-color: #007bff;
                    margin: 10px 0;
                  "
                />
                <h4>{{ $t('admin.csaf_aggregators') }}</h4>
                <b-button
                  size="md"
                  variant="outline-primary"
                  @click="
                    $root.$emit('setModalTitle', $t('admin.add_aggregator'))
                  "
                  v-b-modal.vulnSourceCSAFAddModal
                >
                  <span class="fa fa-plus"></span>
                  {{ $t('admin.add_aggregator') }}
                </b-button>
                <bootstrap-table
                  ref="table_sources"
                  :columns="srcCols"
                  :data="srcData"
                  :options="srcOpts"
                >
                </bootstrap-table>
                <hr
                  style="
                    border: none;
                    height: 1px;
                    background-color: #007bff;
                    margin: 10px 0;
                  "
                />
                <h4>{{ $t('admin.csaf_providers') }}</h4>
                <b-button
                  size="md"
                  variant="outline-primary"
                  @click="
                    $root.$emit('setModalTitle', $t('admin.add_provider'))
                  "
                  v-b-modal.vulnSourceCSAFAddModal
                >
                  <span class="fa fa-plus"></span>
                  {{ $t('admin.add_provider') }}
                </b-button>
                <bootstrap-table
                  ref="table_providers"
                  :columns="provCols"
                  :data="provData"
                  :options="provOpts"
                >
                </bootstrap-table>
                <hr
                  style="
                    border: none;
                    height: 1px;
                    background-color: #007bff;
                    margin: 10px 0;
                  "
                />
                <h4>{{ $t('admin.suggested_discovery_sources') }}</h4>
                <bootstrap-table
                  ref="table_suggested"
                  :columns="recColumns"
                  :data="recData"
                  :options="recOptions"
                  data-click-to-select="true"
                >
                </bootstrap-table>
                <div id="repositoryToolbar" class="bs-table-custom-toolbar">
                  <b-button
                    size="md"
                    variant="outline-primary"
                    @click="markReadSuggestions"
                  >
                    <span class="fa fa-check"></span>
                    {{ $t('admin.mark_selected_read') }}
                  </b-button>
                  <b-button
                    size="md"
                    variant="outline-primary"
                    @click="addSelected"
                  >
                    <span class="fa fa-plus"></span>
                    {{ $t('admin.add_selected') }}
                  </b-button>
                </div>
                <hr />
              </b-card-body>
            </b-tab>
            <b-tab title="CSAF Documents">
              <b-card-body>
                <div id="repositoryToolbar" class="bs-table-custom-toolbar">
                  <!--<h2>{{ $t('admin.csaf_documents') }}:</h2>-->
                  <b-button
                    size="md"
                    variant="outline-primary"
                    v-b-modal.vulnSourceCSAFUpload
                  >
                    <span class="fa fa-upload"></span>
                    {{ $t('admin.upload_file') }}
                  </b-button>
                  <b-button
                    size="md"
                    variant="outline-primary"
                    @click="openCompare"
                  >
                    <span class="fa fa-file"></span>
                    {{ $t('admin.compare_selected') }}
                  </b-button>
                </div>
                <bootstrap-table
                  ref="table_documents"
                  :columns="docColumns"
                  :data="docData"
                  :options="docOpts"
                  data-click-to-select="true"
                >
                </bootstrap-table>
                <div id="repositoryToolbar" class="bs-table-custom-toolbar">
                  <b-button
                    size="md"
                    variant="outline-primary"
                    @click="markReadDocuments"
                  >
                    <span class="fa fa-check"></span>
                    {{ $t('admin.mark_selected_read') }}
                  </b-button>
                  <b-button
                    size="md"
                    variant="outline-primary"
                    @click="deleteSelected"
                  >
                    <span class="fa fa-trash"></span>
                    {{ $t('admin.delete_selected') }}
                  </b-button>
                </div>
              </b-card-body>
            </b-tab>
          </b-tabs>
        </b-card>
      </div>
    </b-card-body>
    <b-card-footer></b-card-footer>
    <vuln-source-c-s-a-f-add v-on:refreshTable="refreshBothCsafSourcesTables" />
    <vuln-source-c-s-a-f-compare
      :leftTitle="compareLeftTitle"
      :rightTitle="compareRightTitle"
      :leftContent="compareLeftContent"
      :rightContent="compareRightContent"
    />
    <vuln-source-c-s-a-f-upload />
    <vuln-source-c-s-a-f-permission />
  </b-card>
</template>
<script>
import Vue from 'vue';
import { Switch as cSwitch } from '@coreui/vue';
import configPropertyMixin from '../mixins/configPropertyMixin';
import VulnSourceCSAFAdd from './VulnSourceCSAFAddModal.vue';
import ActionableListGroupItem from '../../components/ActionableListGroupItem.vue';
import BValidatedInputGroupFormInput from '../../../forms/BValidatedInputGroupFormInput';
import VulnSourceCSAFCompare from './VulnSourceCSAFCompare.vue';
import VulnSourceCSAFUpload from './VulnSourceCSAFUpload.vue';
import VulnSourceCSAFPermission from './VulnSourceCSAFPermission.vue';
import VulnSourceCSAFSourceEdit from './VulnSourceCSAFSourceEdit.vue';
import i18n from '../../../i18n';
import bootstrapTableMixin from '../../../mixins/bootstrapTableMixin';
import EventBus from '../../../shared/eventbus';

export default {
  mixins: [configPropertyMixin, bootstrapTableMixin],
  props: {
    header: String,
  },
  computed: {
    triggerAllTooltip() {
      if (this.triggerAllDisabled && this.triggerAllRemainingMs > 0) {
        const formatted = this.formatRemainingMs(this.triggerAllRemainingMs);
        const translated = this.$t('admin.trigger_all_cooldown', {
          remaining: formatted,
        });
        // If translation is missing, $t often returns the key; provide fallback
        if (
          translated &&
          translated.indexOf('admin.trigger_all_cooldown') === -1
        )
          return translated;
        return `Cooldown: ${formatted}`;
      }
      const hint = this.$t('admin.trigger_all_tooltip');
      if (hint && hint.indexOf('admin.trigger_all_tooltip') === -1) return hint;
      return 'Click to trigger fetching all CSAF sources';
    },
  },
  components: {
    cSwitch,
    VulnSourceCSAFAdd,
    ActionableListGroupItem,
    BValidatedInputGroupFormInput,
    VulnSourceCSAFCompare,
    VulnSourceCSAFUpload,
    VulnSourceCSAFPermission,
    VulnSourceCSAFSourceEdit,
  },
  data() {
    return {
      compareLeftTitle: '',
      compareLeftContent: null,
      compareRightTitle: '',
      compareRightContent: null,
      searchTerm: '',
      configInitialized: false,
      vulnsourceEnabled: false,
      vulnsourceToggleInitialized: false,
      // temporary lockout: disable "Trigger All" button after click (milliseconds)
      triggerAllDisabled: false,
      triggerAllTimer: null,
      // lockout duration for Trigger All (milliseconds). Default 90 seconds.
      // Will be fetched from the server later via config property
      // propertyName: 'extension.csaf.trigger_all_lockout_ms'
      triggerAllLockoutMs: 90 * 1000,
      // trigger all countdown state
      triggerAllExpiry: null,
      triggerAllCountdownTimer: null,
      triggerAllRemainingMs: 0,
      // automatic refresh configuration (milliseconds)
      autoRefreshIntervalMs: 10000,
      autoRefreshTimer: null,
      // track if any row is expanded (edit form is open)
      isAnyRowExpanded: false,
      labelIcon: {
        dataOn: '\u2713',
        dataOff: '\u2715',
      },
      recColumns: [
        {
          title: 'Select',
          field: 'select',
          checkbox: true,
        },
        {
          title: 'ID',
          field: 'id',
          sortable: true,
          visible: false,
        },
        {
          title: 'Name',
          field: 'name',
          sortable: true,
          formatter: (value, row) => {
            return row.seen === false ? `<strong>${value} *</strong>` : value;
          },
        },
        {
          title: 'URL',
          field: 'url',
          class: 'tight',
          sortable: true,
        },
        {
          title: 'Read',
          field: 'seen',
          sortable: true,
        },
        {
          title: 'Actions',
          field: 'actions',
          formatter: (value, row) => {
            return `<button class="btn btn-primary" id="rec-${row.id}"> <span class="fa fa-plus"></span> Add</button>`;
          },
        },
      ],
      recData: [],
      recOptions: {
        search: true,
        showColumns: true,
        showRefresh: true,
        pagination: true,
        sidePagination: 'client',
        queryParamsType: 'pageSize',
        pageList: '[10, 25, 50, 100]',
        pageSize: 10,
        silentSort: false,
        sortName: 'seen',
        sortOrder: 'asc',
        responseHandler: function (res) {
          return res.data || [];
        },
        icons: {
          refresh: 'fa-refresh',
        },
        rowAttributes: (row) => {
          return {
            style: row.new === 'New' ? 'font-weight: bold;' : '',
          };
        },
      },
      docColumns: [
        {
          title: 'Select',
          field: 'select',
          checkbox: true,
        },
        {
          title: 'ID',
          field: 'id',
          sortable: true,
        },
        {
          title: 'Title',
          field: 'title',
          sortable: true,
          formatter: (value, row) => {
            const title =
              row.seen === false ? `<strong>${value} *</strong>` : value;
            // Render a normal anchor with a data attribute; clicks are intercepted and routed client-side
            return `<a href="/advisories/${row.id}" class="csaf-doc-link" data-doc-id="${row.id}">${title}</a>`;
          },
        },
        {
          title: 'Publisher Namespace',
          field: 'publisher',
          sortable: true,
        },
        {
          title: 'Version',
          field: 'version',
          class: 'tight',
          sortable: true,
        },
        {
          title: 'Last fetched',
          field: 'last_fetched',
          class: 'tight',
          sortable: true,
          formatter: (value) => {
            if (!value || value <= 0) {
              return this.$t('admin.never');
            }
            const date = new Date(value);
            return date.toLocaleString();
          },
        },
        {
          title: 'Read',
          field: 'seen',
          sortable: true,
        },
        {
          title: 'Newest Version?',
          field: 'latestVersion',
          formatter: (value, row) => {
            const allRows = this.$refs.table_documents.getData();
            const sameNameDocs = allRows.filter((doc) => doc.name === row.name);
            const formatVersion = (version) =>
              Number(version.replace(/\./g, ''));
            const maxVersion = Math.max(
              ...sameNameDocs.map(
                (doc) => formatVersion(doc.version) || -Infinity,
              ),
            );
            return formatVersion(row.version) === maxVersion ? 'Yes' : 'No';
          },
        },
        // Actions column removed: view via title link
      ],
      docData: [],
      docOpts: {
        search: true,
        showColumns: true,
        showRefresh: true,
        pagination: true,
        sidePagination: 'server',
        dataField: 'advisories',
        responseHandler: function (res, xhr) {
          res.total = xhr.getResponseHeader('X-Total-Count');
          return res;
        },
        queryParamsType: 'pageSize',
        pageList: '[10, 25, 50, 100]',
        pageSize: 10,
        silentSort: false,
        sortName: 'seen',
        sortOrder: 'asc',
        icons: {
          refresh: 'fa-refresh',
        },
      },
      srcCols: [
        {
          title: 'ID',
          field: 'id',
          class: 'tight',
          visible: false,
          sortable: true,
        },
        {
          title: 'Name',
          field: 'name',
          class: 'tight',
          sortable: true,
          width: '180px',
          formatter: (value, row) => {
            return row.new === 'New' ? `${value} *` : value;
          },
        },
        {
          title: 'URL',
          field: 'url',
          class: 'tight',
          sortable: true,
          width: '420px',
        },
        {
          title: 'Last fetched',
          field: 'last_fetched',
          class: 'tight',
          sortable: true,
          formatter: (value) => {
            if (!value || value <= 0) {
              return this.$t('admin.never');
            }
            const date = new Date(value);
            return date.toLocaleString();
          },
        },
        {
          title: this.$t('admin.enabled'),
          field: 'enabled',
          class: 'tight text-center',
          align: 'center',
          width: '80px',
          sortable: true,
          formatter(value, row, index) {
            return value === true
              ? '<i class="fa fa-check-square-o" aria-label="enabled" />'
              : '';
          },
        },
      ],
      srcData: [],
      srcOpts: {
        search: true,
        showColumns: true,
        showRefresh: true,
        pagination: true,
        sidePagination: 'client',
        queryParamsType: 'pageSize',
        pageList: '[10, 25, 50, 100]',
        pageSize: 10,
        silentSort: false,
        responseHandler: function (res) {
          return res.data || [];
        },
        icons: {
          refresh: 'fa-refresh',
        },
        detailView: true,
        detailViewIcon: false,
        detailViewByClick: true,
        onExpandRow: this.vueFormatterInit,
        onCollapseRow: (index, row) => {
          this.isAnyRowExpanded = false;
        },
        onLoadError: (status, res) => {
          if (status === 403) {
            this.$bvModal.show('vulnSourceCSAFPermission');
          }
        },
        detailFormatter: (index, row) => {
          return this.vueFormatter({
            i18n,
            template: `<vuln-source-c-s-a-f-source-edit :source="source" :index="index" source-type="aggregator" />`,
            components: {
              VulnSourceCSAFSourceEdit,
            },
            data() {
              return {
                source: row,
                index: index,
              };
            },
          });
        },
      },
      provCols: [
        {
          title: 'ID',
          field: 'id',
          class: 'tight',
          visible: false,
          sortable: true,
        },
        {
          title: 'Name',
          field: 'name',
          class: 'tight',
          sortable: true,
          width: '180px',
          formatter: (value, row) => {
            return row.new === 'New' ? `${value} *` : value;
          },
        },
        {
          title: 'URL',
          field: 'url',
          class: 'tight',
          sortable: true,
          width: '420px',
        },
        {
          title: 'Last fetched',
          field: 'last_fetched',
          class: 'tight',
          sortable: true,
          formatter: (value) => {
            if (!value || value <= 0) {
              return this.$t('admin.never');
            }
            const date = new Date(value);
            return date.toLocaleString();
          },
        },

        {
          title: this.$t('admin.enabled'),
          field: 'enabled',
          class: 'tight text-center',
          align: 'center',
          width: '80px',
          sortable: true,
          formatter(value, row, index) {
            return value === true
              ? '<i class="fa fa-check-square-o" aria-label="enabled" />'
              : '';
          },
        },
      ],
      provData: [],
      provOpts: {
        search: true,
        showColumns: true,
        showRefresh: true,
        pagination: true,
        sidePagination: 'client',
        queryParamsType: 'pageSize',
        pageList: '[10, 25, 50, 100]',
        pageSize: 10,
        silentSort: false,
        responseHandler: function (res) {
          return res.data || [];
        },
        icons: {
          refresh: 'fa-refresh',
        },
        detailView: true,
        detailViewIcon: false,
        detailViewByClick: true,
        onExpandRow: this.vueFormatterInit,
        onCollapseRow: (index, row) => {
          this.isAnyRowExpanded = false;
        },
        detailFormatter: (index, row) => {
          return this.vueFormatter({
            i18n,
            template: `<vuln-source-c-s-a-f-source-edit :source="source" :index="index" source-type="provider" />`,
            components: {
              VulnSourceCSAFSourceEdit,
            },
            data() {
              return {
                source: row,
                index: index,
              };
            },
          });
        },
      },
    };
  },
  watch: {
    vulnsourceEnabled(newValue) {
      if (!this.vulnsourceToggleInitialized) {
        this.vulnsourceToggleInitialized = true; // skip when initializing
      } else {
        this.updateConfigProperties([
          {
            groupName: 'vuln.datasource',
            propertyName: 'extension.csaf.enabled',
            propertyValue: this.vulnsourceEnabled,
          },
        ]);
      }
    },
  },
  methods: {
    // Override vueFormatterInit from mixin to track row expansion
    vueFormatterInit() {
      this.isAnyRowExpanded = true;
      // Call the mixin's vueFormatterInit
      if (!this.vueFormatters.length) {
        return;
      }
      for (let i = this.vueFormatters.length - 1; i >= 0; i--) {
        const formatter = this.vueFormatters[i];
        if (document.getElementsByClassName(formatter.name)) {
          new Vue(formatter);
          this.vueFormatters.splice(i, 1);
        }
      }
    },
    // showDoc removed: navigation goes to advisory detail page via title link
    handleAdd(id) {
      var addRow = this.$refs.table_suggested
        .getData()
        .find((item) => item.id.toString() === id.toString());
      addRow.discovered = false;
      this.updateCsafSource(addRow);
    },
    addSelected() {
      const selectedRows = this.$refs.table_suggested.getSelections();
      if (selectedRows.length > 0) {
        selectedRows.forEach((item) => {
          item.discovered = false;
          this.updateCsafSource(item);
        });
      }
    },
    async updateCsafSource(prow) {
      let url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_SOURCES}`;
      try {
        const response = await this.axios.post(url, {
          id: prow.id,
          url: prow.url,
          name: prow.name,
          discovered: prow.discovered || false,
          enabled: prow.enabled || false,
          aggregator: prow.aggregator || false,
        });
        this.csafEntry = response.data;
        EventBus.$emit(
          'admin:csafProviders:rowUpdate',
          prow.id,
          this.csafEntry,
        );
        this.$toastr.s(this.$t('message.updated'));
        this.refreshCsafSuggestedTable();
        this.refreshProvidersTable(); // TODO re-evaluate necessity with rowUpdate
      } catch (error) {
        this.$toastr.w(this.$t('condition.unsuccessful_action'));
      }
    },
    async openCompare() {
      const selectedRows = this.$refs.table_documents.getSelections();
      if (selectedRows.length !== 2) {
        alert(this.$t('admin.please_select_two_rows'));
        return;
      }

      const leftDoc = await this.getDocument(selectedRows[0].id);
      const rightDoc = await this.getDocument(selectedRows[1].id);

      this.compareLeftTitle = selectedRows[0].name;
      // Parse the content property which contains the CSAF JSON as a string
      this.compareLeftContent =
        leftDoc && leftDoc.entity && leftDoc.entity.content
          ? JSON.parse(leftDoc.entity.content)
          : null;

      this.compareRightTitle = selectedRows[1].name;
      // Parse the content property which contains the CSAF JSON as a string
      this.compareRightContent =
        rightDoc && rightDoc.entity && rightDoc.entity.content
          ? JSON.parse(rightDoc.entity.content)
          : null;

      this.$bvModal.show('vulnSourceCSAFCompareModal');
    },
    deleteSelected() {
      const selectedRows = this.$refs.table_documents.getSelections();
      if (selectedRows.length === 0) {
        console.log(`nothing selected`);
        return;
      }
      const deletePromises = selectedRows.map((row) => {
        var rowIndex = this.$refs.table_documents
          .getData()
          .findIndex((item) => item.id === row.id);
        const url = `${this.$api.BASE_URL}/${this.$api.URL_ADVISORIES}/${row.id}`;
        return this.axios
          .delete(url)
          .then((response) => {
            EventBus.$emit('admin:csafDocuments:rowDeleted', rowIndex);
            this.$toastr.s(
              this.$t('admin.csaf_document_deleted', { id: row.id }),
            );
          })
          .catch((error) => {
            this.$toastr.w(this.$t('condition.unsuccessful_action'));
          });
      });
    },
    markReadSuggestions() {
      const selectedRows = this.$refs.table_suggested.getSelections();
      if (selectedRows.length > 0) {
        selectedRows.forEach((prow) => {
          prow.seen = true;
          this.updateCsafSource(prow); // TODO issue update on eventbus
        });
      }
      this.refreshCsafSuggestedTable();
    },
    markReadDocuments() {
      const selectedRows = this.$refs.table_documents.getSelections();
      if (selectedRows.length > 0) {
        selectedRows.forEach((row) => {
          // find row index
          var rowIndex = this.$refs.table_documents
            .getData()
            .findIndex((item) => item.id === row.id);

          let url = `${this.$api.BASE_URL}/${this.$api.URL_ADVISORIES}/seen/${row.id}`;
          this.axios
            .post(url, { id: row.id })
            .then((response) => {
              EventBus.$emit(
                'admin:csafDocuments:rowUpdate',
                rowIndex,
                response.data,
              );
              this.$toastr.s(this.$t('admin.csaf_document_read'));
            })
            .catch(() => {
              this.$toastr.w(this.$t('condition.unsuccessful_action'));
            });
        });
      }
    },
    triggerAll() {
      const url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_TRIGGER}/`;
      // disable the button for 90 seconds (temporary workaround)
      try {
        // set disabled flag immediately so UI reflects action
        this.triggerAllDisabled = true;
        // clear any existing timer
        if (this.triggerAllTimer) {
          clearTimeout(this.triggerAllTimer);
          this.triggerAllTimer = null;
        }
        // clear any existing countdown interval
        if (this.triggerAllCountdownTimer) {
          clearInterval(this.triggerAllCountdownTimer);
          this.triggerAllCountdownTimer = null;
        }
        // set expiry timestamp and initialize remaining ms
        const now = Date.now();
        this.triggerAllExpiry = now + this.triggerAllLockoutMs;
        this.triggerAllRemainingMs = this.triggerAllLockoutMs;
        // start 1s interval to update remaining ms for tooltip
        this.triggerAllCountdownTimer = setInterval(() => {
          const remaining = this.triggerAllExpiry - Date.now();
          this.triggerAllRemainingMs = remaining > 0 ? remaining : 0;
          if (remaining <= 0) {
            clearInterval(this.triggerAllCountdownTimer);
            this.triggerAllCountdownTimer = null;
          }
        }, 1000);
        // use configured lockout duration
        this.triggerAllTimer = setTimeout(() => {
          this.triggerAllDisabled = false;
          this.triggerAllTimer = null;
          this.triggerAllExpiry = null;
          this.triggerAllRemainingMs = 0;
        }, this.triggerAllLockoutMs);
      } catch (e) {
        // noop - defensive
      }

      return this.axios
        .post(url)
        .then((response) => {
          this.$toastr.s(this.$t('admin.trigger_all'));
        })
        .catch((error) => {
          this.$toastr.w(this.$t('condition.unsuccessful_action'));
        });
    },
    getDocument(docId) {
      let url = `${this.$api.BASE_URL}/${this.$api.URL_ADVISORIES}/${docId}`;
      return this.axios
        .get(url)
        .then((response) => {
          return response.data;
        })
        .catch((error) => {
          this.$toastr.w(this.$t('condition.unsuccessful_action'));
          return null;
        });
    },
    apiUrl: function () {
      return `${this.$api.BASE_URL}/${this.$api.URL_CSAF_SOURCES}?type=aggregator`;
    },
    apiProvidersUrl: function () {
      return `${this.$api.BASE_URL}/${this.$api.URL_CSAF_SOURCES}?type=provider`;
    },
    apiDocsUrl: function () {
      return `${this.$api.BASE_URL}/${this.$api.URL_ADVISORIES}?format=CSAF`;
    },
    apiDisUrl: function () {
      return `${this.$api.BASE_URL}/${this.$api.URL_CSAF_SOURCES}?discovered=true`;
    },
    refreshBothCsafSourcesTables: function () {
      this.refreshAggregatorsTable();
      this.refreshProvidersTable();
    },
    refreshAggregatorsTable: function () {
      this.$refs.table_sources.refresh({
        url: this.apiUrl(),
        silent: true,
      });
    },
    refreshProvidersTable: function () {
      this.$refs.table_providers.refresh({
        url: this.apiProvidersUrl(),
        silent: true,
      });
    },
    refreshCsafSuggestedTable: function () {
      this.$refs.table_suggested.refresh({
        url: this.apiDisUrl(),
        silent: true,
      });
    },
    refreshCsafDocumentsTable: function () {
      this.$refs.table_documents.refresh({
        url: this.apiDocsUrl(),
        silent: true,
      });
    },
    saveConfiguration: function () {
      this.updateConfigProperties([
        {
          groupName: 'vuln.datasource',
          propertyName: 'extension.csaf.enabled',
          propertyValue: this.vulnsourceEnabled,
        },
      ]);
    },
    // helper to format remaining milliseconds into MM:SS
    formatRemainingMs(ms) {
      if (!ms || ms <= 0) return '0:00';
      const totalSeconds = Math.ceil(ms / 1000);
      const mins = Math.floor(totalSeconds / 60);
      const secs = totalSeconds % 60;
      return `${mins}:${secs.toString().padStart(2, '0')}`;
    },
    startAutoRefresh() {
      // Prevent duplicate timers
      if (this.autoRefreshTimer) return;

      // Immediately refresh once, then schedule periodic refreshes
      if (this.vulnsourceEnabled) {
        this.refreshCsafSuggestedTable();
        this.refreshProvidersTable();
        this.refreshAggregatorsTable();
        this.refreshCsafDocumentsTable();
      }

      // Set up periodic refresh
      this.autoRefreshTimer = setInterval(() => {
        if (!this.vulnsourceEnabled) return;

        // Skip refresh if any row is expanded (edit form is open)
        if (this.isAnyRowExpanded) {
          return;
        }

        try {
          this.refreshCsafSuggestedTable();
          this.refreshProvidersTable();
          this.refreshAggregatorsTable();
          this.refreshCsafDocumentsTable();
        } catch (e) {
          // keep polling even if one refresh fails
          // eslint-disable-next-line no-console
          console.warn('CSAF auto-refresh error', e);
        }
      }, this.autoRefreshIntervalMs);
    },
    stopAutoRefresh() {
      if (this.autoRefreshTimer) {
        clearInterval(this.autoRefreshTimer);
        this.autoRefreshTimer = null;
      }
    },
    updateSourcesTable: function () {
      this.axios.get(this.apiUrl()).then((response) => {
        this.srcData = response;

        this.$toastr.s('Csaf sources updated');
      });
    },
    updateProvidersTable: function () {
      this.axios.get(this.apiProvidersUrl()).then((response) => {
        this.provData = response;
        this.$toastr.s('Csaf providers updated');
      });
    },
    updateDocumentsTable: function () {
      this.axios.get(this.apiUrl()).then((response) => {
        this.docData = response;

        this.$toastr.s('Csaf documents updated');
      });
    },
  },
  mounted() {
    this.axios.get(this.configUrl).then((response) => {
      let configItems = response.data.filter(function (item) {
        return item.groupName === 'vuln.datasource';
      });
      for (let i = 0; i < configItems.length; i++) {
        let item = configItems[i];
        switch (item.propertyName) {
          case 'extension.csaf.enabled':
            if (item.propertyValue === 'true') {
              this.vulnsourceEnabled = true;
            } else {
              this.vulnsourceToggleInitialized = true; // toggle is initialized
            }
            break;
          case 'extension.csaf.trigger_all_lockout_ms':
            // allow server to override default lockout in milliseconds
            const parsed = parseInt(item.propertyValue, 10);
            if (!isNaN(parsed) && parsed > 0) {
              this.triggerAllLockoutMs = parsed;
            }
            break;
        }
      }
      this.refreshBothCsafSourcesTables();
      this.refreshCsafDocumentsTable();
      this.refreshCsafSuggestedTable();
      this.configInitialized = true;
      // start automatic refresh on component load
      this.startAutoRefresh();
    });
    EventBus.$on('admin:csafAggregators:rowDeleted', (index, row) => {
      this.refreshAggregatorsTable();
    });
    EventBus.$on('admin:csafAggregators:rowUpdate', (index, row) => {
      this.$refs.table_sources.updateRow({ index: index, row: row });
      this.$refs.table_sources.expandRow(index);
    });
    EventBus.$on('admin:csafProviders:rowDeleted', (index, row) => {
      this.refreshProvidersTable();
    });
    EventBus.$on('admin:csafProviders:rowUpdate', (index, row) => {
      this.$refs.table_providers.updateRow({ index: index, row: row });
      this.$refs.table_providers.expandRow(index);
    });
    EventBus.$on('admin:csafDocuments:rowUpdate', (index, row) => {
      this.$refs.table_documents.updateRow({ index: index, row: row });
    });
    EventBus.$on('admin:csafDocuments:rowDeleted', (index, row) => {
      this.refreshCsafDocumentsTable();
    });
    EventBus.$on('refreshProvidersTable', () => {
      this.refreshProvidersTable();
    });
    EventBus.$on('refreshAggregatorsTable', () => {
      this.refreshAggregatorsTable();
    });
    EventBus.$on('refreshDocumentsTable', () => {
      this.refreshCsafDocumentsTable();
    });
    this.$refs.table_documents.$el.addEventListener('click', (event) => {
      const target = event.target;
      // handle clicks on title links rendered in the title formatter
      const link = target.closest && target.closest('a.csaf-doc-link');
      if (link) {
        event.preventDefault();
        const docId = link.getAttribute('data-doc-id');
        if (docId) {
          // navigate using vue-router to keep SPA navigation
          this.$router.push({ path: `/advisories/${docId}` });
        }
      }
    });
    this.$refs.table_suggested.$el.addEventListener('click', (event) => {
      const target = event.target;
      if (target.matches('[id^="rec-"]')) {
        const recId = target.id.split('-')[1];
        this.handleAdd(recId);
      }
    });
  },
  beforeDestroy() {
    // stop periodic refresh when component is destroyed
    this.stopAutoRefresh();
    // clear temporary trigger-all timer if present
    if (this.triggerAllTimer) {
      clearTimeout(this.triggerAllTimer);
      this.triggerAllTimer = null;
    }
    if (this.triggerAllCountdownTimer) {
      clearInterval(this.triggerAllCountdownTimer);
      this.triggerAllCountdownTimer = null;
    }
    EventBus.$off('admin:csafAggregators:rowUpdated');
    EventBus.$off('admin:csafAggregators:rowDeleted');
    EventBus.$off('admin:csafProviders:rowUpdated');
    EventBus.$off('admin:csafProviders:rowDeleted');
    EventBus.$off('admin:csafDocuments:rowUpdated');
    EventBus.$off('admin:csafDocuments:rowDeleted');
  },
};
</script>
