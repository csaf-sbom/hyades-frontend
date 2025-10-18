<template>
  <b-modal
    id="vulnSourceCSAFUpload"
    size="md"
    hide-header-close
    no-stacking
    :title="$t('admin.upload_file')"
  >
    <div style="text-align: center">
      <input
        type="file"
        ref="fileInput"
        @change="onFileChange"
        style="display: none"
      />
      <b-validated-input-group-form-input
        id="fileName"
        :label="$t('admin.file_name')"
        input-group-size="mb-3"
        :value="fileName || $t('admin.no_file_selected')"
        readonly
        style="text-align: center"
      />
      <b-button size="md" variant="primary" @click="selectFile">
        {{ $t('admin.please_select_file') }}
      </b-button>
    </div>

    <hr />

    <template v-slot:modal-footer="{ cancel }">
      <b-button size="md" variant="secondary" @click="cancel()">{{
        $t('message.close')
      }}</b-button>
      <b-button
        size="md"
        variant="success"
        @click="upload(cancel)"
        class="ml-2"
        >{{ $t('admin.upload') }}</b-button
      >
    </template>
  </b-modal>
</template>

<script>
import BValidatedInputGroupFormInput from '../../../forms/BValidatedInputGroupFormInput';
import EventBus from '../../../shared/eventbus';

export default {
  name: 'vulnSourceCSAFUpload',
  props: {},
  components: {
    BValidatedInputGroupFormInput,
  },
  data() {
    return {
      selectedFile: null,
      fileName: '',
    };
  },
  methods: {
    selectFile() {
      this.$refs.fileInput.click();
    },
    onFileChange(event) {
      this.selectedFile = event.target.files[0];
      this.fileName = this.selectedFile ? this.selectedFile.name : '';
    },
    async upload(cancel) {
      if (!this.selectedFile) {
        alert(this.$t('admin.please_select_file'));
        return;
      }
      // For now, the only supported upload format is CSAF. Call CSAF-specific upload helper.
      await this.uploadCsaf(cancel);
    },

    async uploadCsaf(cancel) {
      try {
        const formData = new FormData();
        formData.append('file', this.selectedFile);
        // include format=CSAF to indicate CSAF-specific upload
        let url = `${this.$api.BASE_URL}/${this.$api.URL_ADVISORIES}?format=CSAF`;
        const response = await this.axios.post(url, formData, {
          headers: {
            'Content-Type': 'multipart/form-data',
          },
        });
        this.$toastr.s(this.$t('message.updated'));
        EventBus.$emit('refreshDocumentsTable');
        this.resetFile();
        cancel();
      } catch (error) {
        this.$toastr.w(this.$t('condition.unsuccessful_action'));
      }
    },
    resetFile() {
      this.selectedFile = null;
      this.fileName = '';
    },
  },
};
</script>
