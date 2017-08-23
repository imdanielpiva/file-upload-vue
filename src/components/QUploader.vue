<template>
  <div>

    <div
      class="q-uploader"
      v-if="!$props.modal"
    >

      <slot></slot>

    </div>

    <div v-if="$props.modal">

      <q-modal
        ref="modal"
        @open="resizeUploaderModal()"
        :maximized="this.modal === 'maximized'"
        :minimized="this.modal === 'minimized'"
        :modal="$props.modal"
      >

        <div class="q-uploader">

          <slot></slot>

        </div>

      </q-modal>

      <q-btn class="full-width" :color="$props.color" @click="openModal">
        {{ $props.buttonModal }}
      </q-btn>

    </div>

  </div>
</template>

<script>
import {
  QModal,
  QBtn
} from 'quasar'

export default {
  name: 'uploader-body',
  components: {
    QModal,
    QBtn
  },

  props: {
    color: {
      type: String,
      default: 'primary'
    },
    buttonModal: {
      type: String,
      default: 'Modal'
    },
    modal: String,
    modalActionTriggered: Boolean
  },

  watch: {
    modalActionTriggered: function () {
      this.closeModal();
    }
  },

  methods: {
    openModal () {
      this.$refs.modal.open();
    },

    closeModal () {
      this.$refs.modal.close();
    },

    resizeUploaderModal () {
      const modalTypeUser = this.$props.modal;
      const allUploaders = document.querySelectorAll('.q-uploader');
      const allUploadersArray =
      Object
        .keys(allUploaders)
        .map((allUploadersKey) => {
          return allUploaders[allUploadersKey]
        });
      let countUploaders = 0, modalUploader = '';

      allUploadersArray.forEach(() => {
        if (allUploadersArray[countUploaders].parentElement.classList.contains('modal-content')) {
          modalUploader = allUploaders[countUploaders];
        }
        countUploaders += 1;
      });

      if (modalTypeUser === 'maximized' && modalTypeUser !== 'minimized') {
        const modalUploaderParentChildren = modalUploader.parentElement.children;
        const modalUploaderChildrenArray =
        Object
          .keys(modalUploaderParentChildren)
          .map((modalUploaderKey) => {
            return modalUploaderParentChildren[modalUploaderKey]
          });

        const modalFull = modalUploader.parentElement;
        let count = 0, siblingsUploaderModal = 0;

        modalUploaderChildrenArray.forEach(() => {
          siblingsUploaderModal = siblingsUploaderModal + modalUploaderParentChildren[count].offsetHeight;
          count += 1;
        });

        let divDropItemModal = modalUploader.children[1];

        if (divDropItemModal.offsetHeight === 56) {
          divDropItemModal.style.minHeight = (modalFull.offsetHeight - siblingsUploaderModal + divDropItemModal.offsetHeight) + 'px';
        }
      }
    }

  }
}
</script>

<style lang="stylus" scoped>
  .q-uploader-files
    font-size 14px
    min-height 56px
</style>
