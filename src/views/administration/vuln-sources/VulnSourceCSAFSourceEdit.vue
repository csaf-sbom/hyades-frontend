<template>
  <b-row class="expanded-row">
    <b-col sm="12">
      <b-form-group :label="$t('admin.name')">
        <b-form-input type="text" v-model="name" required />
      </b-form-group>
      <b-form-group :label="$t('admin.url')">
        <b-form-input type="url" v-model="url" required />
      </b-form-group>
      <div class="mb-2">
        <c-switch color="primary" v-model="enabled" label v-bind="labelIcon" />
        {{ $t('admin.enabled') }}
      </div>
      <div class="text-right">
        <b-button variant="outline-danger" @click="deleteSource"
          >Delete CSAF source</b-button
        >
        <b-button class="ml-2" variant="outline-danger" @click="resetFetched"
          >Reset last fetched</b-button
        >
        <b-button class="ml-2" variant="outline-primary" @click="updateSource"
          >Save changes</b-button
        >
      </div>
    </b-col>
  </b-row>
</template>

<script>
import { Switch as cSwitch } from '@coreui/vue';
import EventBus from '../../../shared/eventbus';

export default {
  name: 'VulnSourceCSAFSourceEdit',
  components: {
    cSwitch,
  },
  props: {
    source: {
      type: Object,
      required: true,
    },
    index: {
      type: Number,
      required: true,
    },
    sourceType: {
      type: String,
      required: true,
      validator: (value) => ['aggregator', 'provider'].includes(value),
    },
  },
  data() {
    return {
      id: this.source.id,
      name: this.source.name,
      url: this.source.url,
      enabled: this.source.enabled,
      lastFetched: this.source.lastFetched,
      labelIcon: {
        dataOn: '\u2713',
        dataOff: '\u2715',
      },
    };
  },
  computed: {
    apiUrl() {
      return `${this.$api.BASE_URL}/${this.$api.URL_CSAF_SOURCES}`;
    },
    deleteApiUrl() {
      return `${this.apiUrl}/${this.id}`;
    },
    eventPrefix() {
      return this.sourceType === 'aggregator'
        ? 'admin:csafAggregators'
        : 'admin:csafProviders';
    },
    refreshEventName() {
      return this.sourceType === 'aggregator'
        ? 'refreshAggregatorsTable'
        : 'refreshProvidersTable';
    },
  },
  methods: {
    deleteSource() {
      this.axios
        .delete(this.deleteApiUrl)
        .then((response) => {
          EventBus.$emit(`${this.eventPrefix}:rowDeleted`, this.index);
          this.$toastr.s(this.$t('admin.csaf_source_deleted'));
        })
        .catch((error) => {
          this.$toastr.w(this.$t('condition.unsuccessful_action'));
        });
    },
    resetFetched() {
      this.lastFetched = null;
      this.updateSource(true).then(() => {
        EventBus.$emit(this.refreshEventName);
      });
    },
    updateSource(resetLastFetched = false) {
      const payload = {
        id: this.id,
        url: this.url,
        name: this.name,
        enabled: this.enabled,
        aggregator: this.sourceType === 'aggregator',
      };
      
      // Handle lastFetched: either reset to null or send existing value as ISO string
      if (resetLastFetched) {
        payload.lastFetched = null;
      } else if (this.source.lastFetched) {
        // Convert Unix timestamp (seconds) to ISO 8601 date-time string
        const date = new Date(this.source.lastFetched * 1000);
        payload.lastFetched = date.toISOString();
      } else {
        payload.lastFetched = null;
      }
      
      return this.axios
        .post(this.apiUrl, payload)
        .then((response) => {
          EventBus.$emit(
            `${this.eventPrefix}:rowUpdate`,
            this.index,
            response.data,
          );
          this.$toastr.s(this.$t('message.updated'));
        })
        .catch((error) => {
          this.$toastr.w(this.$t('condition.unsuccessful_action'));
        });
    },
  },
};
</script>
