<template lang="pug">
  .page-media-upload

    BContainer
      BRow
        Breadcrumbs(:crumbs="crumbs")
      BRow(fluid)
        BCol
          BForm
            BFormGroup(label="File(s)" label-for="files")
              BFormFile(
                ref="file-upload"
                v-model="files"
                placeholder="Select file(s), or drag and drop here..."
                drop-placeholder="Drop file(s) here!"
                accept=".jpg, .jpeg, .png"
                multiple
              )
            BFormGroup(v-if="hasFiles")
              span {{ files.length }} file{{ (files.length) > 1 ? 's' : '' }} selected
            SpinnerButton.mr-2(
              @click="onSubmit()"
              :disabled="!hasFiles || isUploading"
              :loading="isUploading"
              label="Upload"
              label-when-loading="Uploading"
            )
            BButton(@click="onReset()" variant="danger" :disabled="!hasFiles || isUploading")
              span Reset
</template>

<script>
export default {
  name: 'PageMediaUpload',

  transition: 'fade',

  components: {
    Breadcrumbs: () => import('@/components/layout/Breadcrumbs'),
    SpinnerButton: () => import('@/components/elements/SpinnerButton'),
  },

  data: () => ({
    crumbs: [{ text: 'Media', to: '/media' }, { text: 'Upload', active: true }],
    files: null,
    isUploading: false,
  }),

  computed: {
    hasFiles () {
      return Boolean(this.files != null && this.files.length)
    },
  },

  methods: {
    async onSubmit () {
      const formData = new FormData()
      this.files.forEach((file) => {
        formData.append('files[]', file)
      })
      this.isUploading = true
      const { data } = await this.$axios.post('/api/media', formData)
      if (data.status === 'success') {
        this.$bvToast.toast(
          `${this.files.length} file` +
            `${this.files.length > 1 ? 's' : ''} uploaded successfully!`,
          {
            title: 'Upload Success',
            variant: 'success',
            autoHideDelay: 5000,
          },
        )
        this.files = null
      } else {
        this.$bvToast.toast('Upload failed - please try again', {
          title: 'Error',
          variant: 'danger',
        })
      }
      this.isUploading = false
    },
    onReset () {
      this.files = null
    },
  },
}
</script>
