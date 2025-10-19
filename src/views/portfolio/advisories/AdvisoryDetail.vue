<template>
  <div class="animated fadeIn">
    <!-- Header Card with Title -->
    <b-card :no-body="true" class="mb-3 shadow-sm">
      <b-card-body class="p-4">
        <div class="d-flex align-items-start">
          <div class="advisory-icon mr-3">
            <i class="fa fa-shield fa-2x text-primary"></i>
          </div>
          <div class="flex-grow-1">
            <h4 class="mb-2">{{ advisory.title }}</h4>
            <div class="text-muted">
              <b-badge variant="secondary" class="mr-2">
                <i class="fa fa-tag"></i> {{ advisory.name }}
              </b-badge>
              <b-badge variant="info">
                <i class="fa fa-code-branch"></i> v{{ advisory.version }}
              </b-badge>
            </div>
          </div>
        </div>
      </b-card-body>
    </b-card>

    <b-tabs class="body-bg-color">
      <b-tab :title="$t('admin.overview')">
        <!-- Advisory Information Card -->
        <b-card class="mb-3 shadow-sm">
          <h6 class="card-subtitle mb-3 text-muted">
            <i class="fa fa-info-circle"></i> {{ $t('message.advisory') }} {{ $t('message.details') }}
          </h6>
          <b-row>
            <b-col md="6">
              <dl class="row mb-0">
                <dt class="col-sm-4 text-muted">
                  <i class="fa fa-tag"></i> {{ $t('admin.name') }}
                </dt>
                <dd class="col-sm-8">
                  <strong>{{ advisory.name }}</strong>
                </dd>
                
                <dt class="col-sm-4 text-muted">
                  <i class="fa fa-code-branch"></i> {{ $t('admin.version') }}
                </dt>
                <dd class="col-sm-8">{{ advisory.version }}</dd>
              </dl>
            </b-col>
            <b-col md="6">
              <dl class="row mb-0">
                <dt class="col-sm-4 text-muted">
                  <i class="fa fa-clock-o"></i> {{ $t('admin.last_fetched') }}
                </dt>
                <dd class="col-sm-8">{{ formatDate(advisory.lastFetched) }}</dd>
                
                <dt class="col-sm-4 text-muted">
                  <i class="fa fa-link"></i> {{ $t('admin.url') }}
                </dt>
                <dd class="col-sm-8">
                  <a :href="advisory.url" target="_blank" class="text-truncate d-inline-block" style="max-width: 300px;">
                    {{ advisory.url }} <i class="fa fa-external-link"></i>
                  </a>
                </dd>
              </dl>
            </b-col>
          </b-row>
        </b-card>

        <!-- Publisher Information Card -->
        <b-card class="mb-3 shadow-sm">
          <h6 class="card-subtitle mb-3 text-muted">
            <i class="fa fa-building"></i> {{ $t('admin.publisher') }}
          </h6>
          <b-row>
            <b-col md="4">
              <dl class="mb-0">
                <dt class="text-muted small">{{ $t('admin.name') }}</dt>
                <dd><strong>{{ doc.document.publisher.name }}</strong></dd>
              </dl>
            </b-col>
            <b-col md="4">
              <dl class="mb-0">
                <dt class="text-muted small">{{ $t('admin.namespace') }}</dt>
                <dd><code>{{ doc.document.publisher.namespace }}</code></dd>
              </dl>
            </b-col>
            <b-col md="4">
              <dl class="mb-0">
                <dt class="text-muted small">{{ $t('admin.category') }}</dt>
                <dd>
                  <b-badge variant="primary">{{ doc.document.publisher.category }}</b-badge>
                </dd>
              </dl>
            </b-col>
          </b-row>
        </b-card>
        <!-- Notes Cards -->
        <b-card
          v-for="(value, key) in doc.document.notes"
          :key="key"
          class="mb-3 shadow-sm"
        >
          <h6 class="card-subtitle mb-2">
            <i class="fa fa-file-text-o"></i> {{ value.category }}
          </h6>
          <p class="card-text mb-0" style="white-space: pre-wrap;">{{ value.text }}</p>
        </b-card>

        <!-- Statistics Card -->
        <b-card class="mb-3 shadow-sm">
          <h6 class="card-subtitle mb-3 text-muted">
            <i class="fa fa-bar-chart"></i> {{ $t('admin.statistics') }}
          </h6>
          <b-row>
            <b-col md="6">
              <div class="stat-box text-center p-3 mb-3 mb-md-0">
                <div class="stat-icon text-primary mb-2">
                  <i class="fa fa-cube fa-3x"></i>
                </div>
                <div class="stat-value h2 mb-1">{{ nProjects }}</div>
                <div class="stat-label text-muted">{{ $t('admin.affected_projects') }}</div>
              </div>
            </b-col>
            <b-col md="6">
              <div class="stat-box text-center p-3">
                <div class="stat-icon text-success mb-2">
                  <i class="fa fa-cubes fa-3x"></i>
                </div>
                <div class="stat-value h2 mb-1">{{ nComponents }}</div>
                <div class="stat-label text-muted">{{ $t('admin.affected_components') }}</div>
              </div>
            </b-col>
          </b-row>
        </b-card>
        <!-- JSON View Toggle -->
        <div class="mt-3 d-flex justify-content-end">
          <b-button
            size="sm"
            variant="outline-secondary"
            @click="showJson = !showJson"
          >
            <i :class="showJson ? 'fa fa-eye-slash' : 'fa fa-eye'"></i>
            {{ showJson ? $t('admin.hide_json') : $t('admin.show_json') }}
          </b-button>
        </div>
        <b-collapse v-model="showJson">
          <b-card class="mt-3 shadow-sm">
            <div class="d-flex justify-content-between align-items-center mb-2">
              <h6 class="mb-0"><i class="fa fa-code"></i> Raw JSON Document</h6>
              <b-button size="sm" variant="outline-primary" @click="copyJson">
                <i class="fa fa-clipboard"></i> Copy
              </b-button>
            </div>
            <pre class="json-viewer mb-0">{{ JSON.stringify(doc, null, 2) }}</pre>
          </b-card>
        </b-collapse>
      </b-tab>
      <b-tab :title="$t('admin.affected_projects')">
        <bootstrap-table
          ref="table_projects"
          :columns="projectsColumns"
          :data="affectedProjects"
          :options="options"
        />
      </b-tab>
      <b-tab :title="$t('admin.product_tree')">
        <b-card>
          <csaf-product-tree v-if="doc" :content="doc" />
        </b-card>
      </b-tab>
      <b-tab :title="$t('admin.vulnerabilities')">
        <bootstrap-table
          ref="table_vulnerabilities"
          :columns="vulnerabilitiesColumns"
          :data="vulnerabilities"
          :options="options"
        />
      </b-tab>
      <b-tab :title="$t('admin.document_history')">
        <bootstrap-table
          ref="table_history"
          :columns="historyColumns"
          :data="historyData"
          :options="historyOptions"
        />
        <!--  <b-card
          :title="value.date"
          v-for="(value, key) in doc.document.tracking.revision_history"
          :key="key"
        >
          <b-card-text>
            {{ value.summary }}
          </b-card-text>
        </b-card> -->
      </b-tab>
    </b-tabs>
  </div>
</template>

<script>
import xssFilters from 'xss-filters';
import common from '../../../shared/common';
import EventBus from '../../../shared/eventbus';
import CsafProductTree from '../../administration/vuln-sources/CsafProductTree.vue';

export default {
  props: {
    advisoryId: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      showJson: false,
      nProjects: 0,
      nComponents: 0,
      advisory: {},
      doc: {},
      projectsColumns: [
        {
          title: this.$t('admin.name'),
          field: 'name',
          sortable: true,
          formatter(value, row, index) {
            let url = xssFilters.uriInUnQuotedAttr(
              '../projects/' + row.uuid + '/advisories',
            );
            return `<a href="${url}">${xssFilters.inHTMLData(value)}</a>`;
          },
        },
        {
          title: this.$t('admin.version'),
          field: 'version',
          class: 'tight',
          sortable: true,
          width: '350px',
        },
        {
          title: this.$t('admin.description'),
          field: 'desc',
          align: 'center',
          sortable: true,
        },
      ],
      vulnerabilitiesColumns: [
        {
          title: this.$t('message.name'),
          field: 'vulnId',
          sortable: true,
          formatter(value, row, index) {
            let url = xssFilters.uriInUnQuotedAttr(
              '../vulnerabilities/' +
                row.source +
                '/' +
                encodeURIComponent(value),
            );
            return (
              common.formatSourceLabel(row.source) +
              ` <a href="${url}">${xssFilters.inHTMLData(value)}</a>`
            );
          },
        },
        {
          title: this.$t('message.title'),
          field: 'title',
          sortable: true,
        },
        {
          title: this.$t('message.severity'),
          field: 'severity',
          class: 'tight',
          align: 'center',
          sortable: true,
          width: '150px',
          formatter(value, row, index) {
            return common.formatSeverityLabel(value);
          },
        },
      ],
      affectedProjects: [],
      vulnerabilities: [],
      historyColumns: [
        {
          //date
          title: this.$t('message.date'),
          field: 'date',
          sortable: true,
          formatter: (cell) => {
            const date = new Date(cell);
            const options = {
              year: 'numeric',
              month: '2-digit',
              day: '2-digit',
              hour: '2-digit',
              minute: '2-digit',
              hour12: false,
            };
            return date.toLocaleString('de-DE', options).replace(',', '');
          },
        },
        {
          //version
          title: this.$t('admin.version'),
          field: 'version',
          class: 'tight',
          sortable: true,
          width: '350px',
        },
        {
          //summary
          title: this.$t('message.summary'),
          field: 'summary',
          align: 'center',
          sortable: true,
        },
      ],
      historyData: [],
      historyOptions: {
        search: true,
        showColumns: true,
        showRefresh: true,
        pagination: true,
        sidePagination: 'client',
        queryParamsType: 'pageSize',
        pageList: '[10, 25, 50, 100]',
        pageSize: 10,
        silentSort: false,
        sortName: 'version',
        sortOrder: 'desc',
        icons: {
          refresh: 'fa-refresh',
        },
      },
      options: {
        search: true,
        showColumns: true,
        showRefresh: true,
        pagination: true,
        sidePagination: 'client',
        queryParamsType: 'pageSize',
        pageList: '[10, 25, 50, 100]',
        pageSize: 10,
        silentSort: false,
        sortName: 'name',
        sortOrder: 'asc',
        icons: {
          refresh: 'fa-refresh',
        },
      },
    };
  },
  components: {
    CsafProductTree,
  },
  methods: {
    formatDate(value) {
      const date = new Date(value * 1000);
      return date.toLocaleString();
    },
    copyJson() {
      const jsonText = JSON.stringify(this.doc, null, 2);
      navigator.clipboard.writeText(jsonText).then(() => {
        this.$toasted.show('JSON copied to clipboard', {
          type: 'success',
          icon: 'clipboard',
        });
      }).catch(() => {
        this.$toasted.show('Failed to copy JSON', {
          type: 'error',
        });
      });
    },
    apiUrl: function () {
      let url = `${this.$api.BASE_URL}/${this.$api.URL_ADVISORIES}/${this.advisoryId}`;
      return url;
    },
    loadData: function () {
      if (!this.advisoryId) {
        return;
      }
      this.axios.get(this.apiUrl()).then((response) => {
        this.advisory = response.data.entity;
        this.nProjects = response.data.affectedProjects.length;
        this.nComponents = response.data.numAffectedComponents;
        this.doc = JSON.parse(response.data.entity.content);
        this.affectedProjects = response.data.affectedProjects;
        this.vulnerabilities = response.data.vulnerabilities;
        EventBus.$emit('addCrumb', this.advisory.name);
        this.$title = this.advisory.name;
        this.historyData = this.doc.document.tracking.revision_history.map(
          (item) => ({
            date: item.date,
            version: item.number,
            summary: item.summary,
          }),
        );
      });
    },
    routeTo(path) {
      if (path) {
        if (
          !this.$route.fullPath.toLowerCase.includes('/' + path.toLowerCase())
        ) {
          // TODO enable tab routing
        }
      }
    },
  },
  watch: {
    advisoryId(newValue) {
      EventBus.$emit('crumble');
      this.loadData();
    },
    $route() {
      //this.getTabFromRoute().activate();
    },
  },
  mounted() {
    this.loadData();
  },
  destroyed() {
    EventBus.$emit('crumble');
  },
};
</script>

<style scoped></style>
