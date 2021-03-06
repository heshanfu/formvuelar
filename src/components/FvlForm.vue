<template>
  <form
    :class="{'fvl-form-drag-over': isDragging}"
    @submit.prevent="submit()"
    @dragover="isDragging=true"
    @dragend="isDragging=false"
    @dragleave="isDragging=false"
    @dragexit="isDragging=false"
    @drop="isDragging=false"
  >
    <slot></slot>
  </form>
</template>

<script>
  import axios from 'axios'
  export default {
    props: {
      method: {
        type: String,
        default: 'post',
        validator: function(value) {
          // The value must match one of the axios methods
          return ['get', 'post', 'put', 'patch', 'delete'].indexOf(value) !== -1
        }
      },
      url: {
        type: String,
        required: true
      },
      multipart: {
        type: Boolean,
        default: false
      },
      data: {
        type: Object,
        default: () => {
          return {}
        }
      }
    },
    data() {
      return {
        errors: {},
        uploadPercentage: 0,
        isLoading: false,
        isDragging: false
      }
    },

    methods: {
      prepareData() {
        let rawData = this.data
        let formData = new FormData()

        // Object.keys(rawData).forEach(function(field) {
        //   // Append files of each single file input
        //   if (field instanceof File) {
        //     Object.keys(field).forEach(function(key) {
        //       formData.append(key, rawData.file[key])
        //     })
        //   }
        // })
        // Append files of each multi file input
        // if (typeof rawData.files !== 'undefined') {
        //   Object.keys(rawData.files).forEach(function(fileField) {
        //     // Iterate over any file sent over appending the files to the form data.
        //     for (var i = 0; i < fileField.length; i++) {
        //       let file = fileField[i]
        //       formData.append(fileField + '[' + i + ']', file)
        //     }
        //   })
        // }

        // Map remaining data into formData
        // Object.keys(rawData.file).map(file => {
        //     formData.append('file', file);
        //     /*
        //     Iteate over any file sent over appending the files
        //     to the form data.
        //     */
        //     // for( var i = 0; i < files.length; i++ ){
        //     // let file = files[i];

        //     // formData.append('files[' + i + ']', file);
        //     // }
        // });

        //delete rawData['files'];
        // Map remaining data into formData
        Object.keys(rawData).map(e => {
          formData.append(e, rawData[e])
        })

        for (var pair of formData.entries()) {
          console.log(pair[0] + ', ' + pair[1])
        }
        return formData
      },

      submit() {
        let $this = this
        this.isLoading = true
        axios({
          method: this.method,
          url: this.url,
          data: this.multipart ? this.prepareData() : this.data,
          // only set params if request is get or delete
          params: this.method == 'get' || this.method == 'delete' ? this.data : {},
          headers: this.multipart
            ? {
                'Content-Type': 'multipart/form-data'
              }
            : {},

          // Calculate current upload percentage
          onUploadProgress: function(progressEvent) {
            let percentage = Math.round((progressEvent.loaded * 100) / progressEvent.total)
            $this.uploadPercentage = percentage
            $this.$emit('uploadProgress', percentage)
          }
        })
          .then(function(response) {
            $this.$emit('success', response)
          })
          .catch(function(error) {
            /* Catch validation errors if status is 422 */
            if (error.response && error.response.status === 422) {
              $this.errors = error.response.data.errors
            }
            $this.$emit('error', error)
          })
          .then(function() {
            $this.isLoading = false
            $this.uploadPercentage = 0
            $this.$emit('uploadProgress', 0)
          })
      },

      dirty(name) {
        this.errors[name] = false
      },

      getErrors(name) {
        return this.errors[name] ? this.errors[name] : []
      },

      hasErrors(name) {
        return this.errors[name] && this.errors[name] !== [] ? true : false
      }
    }
  }
</script>

