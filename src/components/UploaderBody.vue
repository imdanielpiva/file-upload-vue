<template>
  <div>

    <uploader-body
      :modal="$props.modal"
      :color="$props.color"
      :buttonModal="$props.buttonModal"
      :modalActionTriggered="this.modalActionTriggered"
    >

      <q-input-frame
        ref="input"
        class="no-margin"
        :prefix="prefix"
        :suffix="suffix"
        :stack-label="stackLabel"
        :float-label="floatLabel"
        :error="error"
        :disable="disable"
        inverted
        :before="before"
        :after="after"
        :color="color"
        :align="align"
        :length="length"
        additional-length
      >

        <q-icon
          v-if="$props.modal"
          slot="before"
          name="arrow back"
          class="q-uploader-pick-button q-if-control relative-position overflow-hidden"
          @click="changeModalStats"
          :disabled="addDisabled"
        ></q-icon>

        <div
          class="col row items-center q-input-target"
          v-html="label"
        ></div>

        <q-spinner
          v-if="uploading"
          slot="after"
          size="24px"
          class="q-if-control"
        ></q-spinner>

        <q-icon
          v-if="!uploading"
          slot="after"
          name="add"
          class="q-uploader-pick-button q-if-control relative-position overflow-hidden"
          @click="__pick"
          :disabled="addDisabled"
        >
          <input
            type="file"
            ref="file"
            class="q-uploader-input absolute-full"
            title="Selecionar arquivos para fazer o upload"
            :accept="accept"
            :multiple="multiple"
            @change="__add"
          >
        </q-icon>

        <q-icon
          v-if="!hideUploadButton && !uploading"
          slot="after"
          name="cloud_upload"
          class="q-if-control"
          :disabled="length === 0 || allFilesUploaded"
          @click="upload"
        ></q-icon>

      </q-input-frame>

      <div
        ref="dropItemsUploader"
        class="q-uploader-files scroll"
        :class="[ 'bg-' + $props.dropColor ]"
        :style="{ minHeight: transformHeight($props.height) - 50 + 'px' }"
        @dragover="handleDragOver"
        @drop="handleFileSelect"
      >

        <q-item
          v-for="file in files"
          :key="file.name"
          class="q-uploader-file"
        >
          <q-progress v-if="!hideUploadProgress"
            class="q-uploader-progress-bg absolute-full"
            :color="file.__failed ? 'negative' : $props.progressColor"
            :percentage="file.__progress"
          ></q-progress>

          <div class="q-uploader-progress-text absolute" v-if="!hideUploadProgress">
            {{ file.__progress }}%
          </div>

          <q-item-side v-if="file.__img" :image="file.__img.src"></q-item-side>
          <q-item-side v-else icon="insert_drive_file" :color="color"></q-item-side>

          <q-item-main :label="file.name" :sublabel="file.__size"></q-item-main>

          <q-item-side right>
            <q-item-tile
              icon="clear"
              :color="color"
              class="cursor-pointer"
              @click="__remove(file)"
            ></q-item-tile>
          </q-item-side>
        </q-item>
      </div>

    </uploader-body>

  </div>
</template>

<script>
import {
  format,
  QSpinner,
  QIcon,
  QProgress,
  QItem,
  QItemSide,
  QItemMain,
  QItemTile
} from 'quasar'

import FrameMixin from './input-frame/input-frame-mixin'

import QInputFrame from './input-frame/QInputFrame.vue'

import UploaderBody from './QUploader.vue'

const { humanStorageSize } = format;

function initFile (file) {
  file.__doneUploading = false
  file.__failed = false
  file.__uploaded = 0
  file.__progress = 0
};

export default {
  name: 's-file-upload',
  mixins: [FrameMixin],
  components: {
    QInputFrame,
    QSpinner,
    QIcon,
    QProgress,
    QItem,
    QItemSide,
    QItemMain,
    QItemTile,
    UploaderBody
  },

  props: {
    name: {
      type: String,
      default: 'file'
    },
    additionalFields: {
      type: Array,
      default: () => []
    },
    color: {
      type: String,
      default: 'primary'
    },
    dropColor: {
      type: String,
      default: 'light-blue-3'
    },
    progressColor: {
      type: String,
      default: 'light-blue-10'
    },
    buttonModal: {
      type: String,
      default: 'Modal'
    },
    accept: String,
    multiple: Boolean,
    hideUploadButton: Boolean,
    hideUploadProgress: Boolean,
    noThumbnails: Boolean,
    uploader: Object,
    height: String,
    headers: Object,
    modal: String
  },

  data () {
    return {
      queue: [],
      files: [],
      uploading: false,
      uploadedSize: 0,
      totalSize: 0,
      modalActionTriggered: false,
      allFilesUploaded: false,
      filesUploaded: []
    };
  },

  mounted () {
    const uploaders = document.querySelectorAll('.q-uploader'), propHeight = this.$props.height;
    let countUploaders = 0, filteredUploader = '', attrModalFromUploaders = '';

    uploaders.forEach(() => {
      attrModalFromUploaders = uploaders[countUploaders].parentElement.parentElement.getAttribute('modal');
      if (attrModalFromUploaders === null) {
        filteredUploader = uploaders[countUploaders];
      }
      countUploaders += 1;
    });

    if (filteredUploader !== '' && !propHeight) {
      let heightUploader = 106, countPageContent = 0, heightOfPageContents = 0;
      const dropItems = this.$refs.dropItemsUploader;
      const headerUploader = filteredUploader.offsetHeight - dropItems.offsetHeight;
      const pageRoot = document.querySelector('#q-app').children[0];
      const pageContents = pageRoot.children;
      const pageContentsArray =
      Object
        .keys(pageContents)
        .map((pageContentsKey) => {
          return pageContents[pageContentsKey]
        });

      dropItems.style.maxHeight = 'none';

      pageContentsArray.forEach(() => {
        heightOfPageContents = heightOfPageContents + pageContentsArray[countPageContent].offsetHeight;
        countPageContent += 1;
      });

      while (heightUploader <= 106) {
        if (heightUploader === 106) {
          filteredUploader = filteredUploader.parentElement;
          if (filteredUploader.nodeName === 'BODY') {
            return;
          }
          heightUploader = filteredUploader.offsetHeight;
        }
      }

      const uploaderChildren = filteredUploader.children;
      const uploaderChildrenArray =
      Object
        .keys(uploaderChildren)
        .map((uploaderChildrenKey) => {
          return uploaderChildren[uploaderChildrenKey] }
        );

      if (uploaderChildrenArray.length <= 1) {
        return dropItems.style.minHeight = heightUploader - headerUploader + 'px';
      }

      let countUploaderChildren = 0, heightOfUploaderChildren = 0;
      if (uploaderChildrenArray.length > 1) {
        uploaderChildrenArray.forEach(() => {
          heightOfUploaderChildren = heightOfUploaderChildren + uploaderChildren[countUploaderChildren].offsetHeight;
          countUploaderChildren += 1;
        });

        return dropItems.style.minHeight = ((filteredUploader.offsetHeight - heightOfUploaderChildren) + dropItems.offsetHeight) + 'px';
      }
    }
  },

  computed: {
    length () {
      return this.queue.length;
    },

    label () {
      const total = humanStorageSize(this.totalSize);
      return this.uploading
        ? `${(this.progress).toFixed(2)}% (${humanStorageSize(this.uploadedSize)} / ${total})`
        : `${this.length} (${total})`;
    },

    progress () {
      return this.totalSize ? Math.min(99.99, this.uploadedSize / this.totalSize * 100) : 0;
    },

    addDisabled () {
      return !this.multiple && this.length >= 1;
    }
  },

  methods: {
    changeModalStats () {
      this.modalActionTriggered = !this.modalActionTriggered;
    },

    transformHeight (height) {
      if (this.$props.height) {
        if (!this.$props.modal) {
          let declaredHeight = parseInt(height, 10);
          const declaredHeightLength = declaredHeight.toString().length;
          const unitType = height.slice(declaredHeightLength);

          if (unitType === 'px') {}

          else if (unitType === 'vh' || unitType === '%') {
            declaredHeight = (innerHeight / 100) * declaredHeight;
          }

          else if (unitType === 'cm') {
            declaredHeight = declaredHeight * 37.795275593333;
          }

          else {
            declaredHeight = 106;
          }

          return declaredHeight > 106 ? declaredHeight : 106;
        }
      }

      return 106;
    },

    handleFileSelect (evt) {
      evt.stopPropagation();
      evt.preventDefault();
      this.__add(evt);
    },

    handleDragOver (evt) {
      evt.stopPropagation();
      evt.preventDefault();
      evt.dataTransfer.dropEffect = 'copy';
    },

    __add (e) {
      if (this.addDisabled) {
        return;
      }

      this.allFilesUploaded = false;

      let files = '';

      if (e.type === 'change') {
        files = Array.prototype.slice.call(e.target.files);
      }

      if (e.type === 'drop') {
        files = Array.prototype.slice.call(e.dataTransfer.files);
      }

      let fileTypes = (this.$props.accept).split(', ');

      let indexFileMap = -1;
      files = files.map(file => {
        indexFileMap += 1;
        return fileTypes.includes(files[indexFileMap].type) ? file : undefined;
      });

      files = files.filter(function (type) { return type !== undefined });

      files = files
      .filter(file => !this.queue.some(f => file.name === f.name))
      .map(file => {
        initFile(file);

        file.__size = humanStorageSize(file.size);

        if (this.noThumbnails || !file.type.startsWith('image')) {
          this.queue.push(file);
        }
        else {
          const reader = new FileReader();
          reader.onload = (e) => {
            let img = new Image();
            img.src = e.target.result;
            file.__img = img;
            this.queue.push(file);
            this.__computeTotalSize();
          }
          reader.readAsDataURL(file);
        }

        return file;
      });

      this.files = this.files.concat(files);
      this.$emit('add', files);
      this.__computeTotalSize();
    },

    __computeTotalSize () {
      this.totalSize = this.length
        ? this.queue.map(f => f.size).reduce((total, size) => total + size)
        : 0;
    },

    __remove (file) {
      const
        name = file.name,
        done = file.__doneUploading;

      this.$emit(`remove:${done ? 'done' : 'cancel'}`, file, file.xhr);

      if (!done) {
        this.queue = this.queue.filter(obj => obj.name !== name);
      }

      file.__removed = true;
      this.files = this.files.filter(obj => obj.name !== name);
      this.__computeTotalSize();
    },

    __pick () {
      if (!this.addDisabled && this.$q.platform.is.mozilla) {
        this.$refs.file.click();
      }
    },

    __getUploadPromise (file) {
      const form = new FormData();

      try {
        form.append('Content-Type', file.type || 'application/octet-stream');
        form.append(this.name, file);
        this.additionalFields.forEach(field => {
          form.append(field.name, field.value);
        });
      }
      catch (e) {
        return;
      }

      initFile(file);

      return new Promise((resolve, reject) => {
        const upload = this.$props.uploader.saveOnlyRemote();

        upload.on('progress', ({ progress, done, cancel }) => {
          if (done) {
            file.__doneUploading = true;
            file.__progress = 100;
            this.$emit('uploaded', file, upload);
            resolve(file);
          }

          if (file.__removed) {
            this.uploadedSize -= file.__uploaded;
            file.__failed = true;
            file.__removed = false;
            this.$emit('fail', file, upload);
            reject(upload);
          }

          if (file.__removed === undefined) {
            let uploaded = (file.size / 100) * progress;
            this.uploadedSize += uploaded - file.__uploaded;
            file.__uploaded = uploaded;
            file.__progress = parseInt(progress, 10);
          }
        });

        upload.on('error', (err) => {
          file.__failed = true;
          this.$emit('fail', file, upload);
          reject(upload);
        });
      })
    },

    upload () {
      this.queue = this.queue.filter(file => !file.__doneUploading);
      this.__computeTotalSize();

      const length = this.length;
      if (length === 0) {
        return;
      }

      let filesDone = 0;
      this.uploadedSize = 0;
      this.uploading = true;
      this.$emit('start');

      let solved = () => {
        filesDone++;
        if (filesDone === length) {
          let countFilesUploaded = 0;
          let filesUploadedNow = this.queue.filter(file => file.__doneUploading);
          filesUploadedNow.forEach(() => {
            this.filesUploaded.push(filesUploadedNow[countFilesUploaded]);
          });
          this.uploading = false;
          this.allFilesUploaded = true;
          this.__computeTotalSize();
          this.$emit('finish');
        }
      }

      this.queue
      .map(file => this.__getUploadPromise(file))
      .forEach(promise => {
        promise.then(solved).catch(solved);
      });
    }
  }
}
</script>

<style lang="stylus" scoped>
  .q-uploader-files
    font-size 14px
    min-height 56px

  input[type=file], input[type=file]::-webkit-file-upload-button
    cursor pointer !important
</style>
