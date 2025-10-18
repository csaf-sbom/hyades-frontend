<template>
  <div class="animated fadeIn">
    <b-card :no-body="true">
      <b-card-body class="p-3 clearfix">
        <div class="h5 mb-0 mt-2">{{ advisory.name }}</div>
      </b-card-body>
    </b-card>

    <b-tabs class="body-bg-color">
      <b-tab :title="$t('admin.overview')">
        <b-card>
          <table>
            <tr>
              <th>{{ $t('admin.name') }}</th>
              <td>
                {{ advisory.name }}
              </td>
            </tr>
            <tr>
              <th>{{ $t('admin.tracking_id') }}</th>
              <td>{{ advisory.trackingID }}</td>
            </tr>
            <tr>
              <th>{{ $t('admin.version') }}</th>
              <td>{{ advisory.trackingVersion }}</td>
            </tr>
            <tr>
              <th>{{ $t('admin.last_fetched') }}</th>
              <td>{{ formatDate(advisory.lastFetched) }}</td>
            </tr>
            <tr>
              <th>{{ $t('admin.url') }}</th>
              <td>
                <a :href="advisory.url" target="_blank">{{ advisory.url }}</a>
              </td>
            </tr>
          </table>
        </b-card>
        <b-card :title="$t('admin.publisher')">
          <table>
            <tr>
              <th>{{ $t('admin.name') }}</th>
              <td>
                {{ doc.document.publisher.name }}
              </td>
            </tr>
            <tr>
              <th>{{ $t('admin.namespace') }}</th>
              <td>
                {{ doc.document.publisher.namespace }}
              </td>
            </tr>
            <tr>
              <th>{{ $t('admin.category') }}</th>
              <td>
                {{ doc.document.publisher.category }}
              </td>
            </tr>
          </table>
        </b-card>
        <b-card
          :title="value.category"
          v-for="(value, key) in doc.document.notes"
          :key="key"
        >
          <b-card-text>
            {{ value.text }}
          </b-card-text>
        </b-card>
        <b-card :title="$t('admin.statistics')">
          <table>
            <tr>
              <th>Affected Projects</th>
              <td>{{ nProjects }}</td>
            </tr>
            <tr>
              <th>Affected Components</th>
              <td>
                {{ nComponents }}
              </td>
            </tr>
          </table>
        </b-card>
        <div class="mt-2 d-flex justify-content-end">
          <b-button
            size="sm"
            variant="outline-secondary"
            @click="showJson = !showJson"
          >
            {{ showJson ? $t('admin.hide_json') : $t('admin.show_json') }}
          </b-button>
        </div>
        <b-collapse v-model="showJson">
          <b-card class="mt-2">
            <pre class="mb-0">{{ JSON.stringify(doc, null, 2) }}</pre>
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
  methods: {
    formatDate(value) {
      const date = new Date(value * 1000);
      return date.toLocaleString();
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
