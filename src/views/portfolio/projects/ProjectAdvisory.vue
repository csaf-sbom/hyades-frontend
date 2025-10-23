<template>
  <bootstrap-table
    ref="tableAdvisoryFindings"
    :columns="taColumns"
    :data="taData"
    :options="taOptions"
  />
</template>

<script>
import BootstrapToggle from 'vue-bootstrap-toggle';
import permissionsMixin from '@/mixins/permissionsMixin';
import xssFilters from 'xss-filters';

export default {
  props: {
    row: Object,
    projectUuid: String,
  },
  data() {
    return {
      taColumns: [
        {
          title: this.$t('admin.component'),
          field: 'name',
          sortable: true,
          formatter: (value, row, index) => {
            let url = xssFilters.uriInUnQuotedAttr(
              '../../../components/' + row.componentUuid,
            );
            let dependencyGraphUrl = xssFilters.uriInUnQuotedAttr(
              '../../../projects/' +
                this.projectUuid +
                '/dependencyGraph/' +
                row.componentUuid,
            );
            return (
              `<a href="${dependencyGraphUrl}"<i class="fa fa-sitemap" aria-hidden="true" style="float:right; padding-top: 4px; cursor:pointer" data-toggle="tooltip" data-placement="bottom" title="Show in dependency graph"></i></a> ` +
              `<a href="${url}">${xssFilters.inHTMLData(value)}</a>`
            );
          },
        },
        {
          title: this.$t('admin.version'),
          field: 'version',
          class: 'tight',
          sortable: true,
        },
        {
          title: this.$t('admin.group'),
          field: 'group',
          sortable: true,
        },
        {
          title: this.$t('admin.confidence'),
          field: 'confidence',
          sortable: true,
          formatter: (cell) => {
            return `${cell}%`;
          },
        },
      ],
      taData: [],
      taOptions: {
        search: false,
        showColumns: true,
        showRefresh: false,
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
  watch: {},
  mixins: [permissionsMixin],
  methods: {
    apiUrl: function () {
      let url = `${this.$api.BASE_URL}/${this.$api.URL_ADVISORIES}/project/${this.projectUuid}/advisory/${this.row.documentId}`;
      return url;
    },
    refreshTable: function () {
      this.$refs.tableAdvisoryFindings.refresh({
        url: this.apiUrl(),
        pageNumber: 1,
        silent: true,
      });
    },
  },
  mounted() {
    this.refreshTable();
  },
  components: {
    BootstrapToggle,
  },
};
</script>
