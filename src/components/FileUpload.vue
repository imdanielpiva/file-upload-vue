<template>
  <div>

    <s-file-upload
      color="teal-10"
      dropColor="teal-3"
      progressColor="teal-10"
      float-label="Cadastro de Arquivos"
      accept="image/jpeg, image/jpg, image/png, image/gif"
      multiple
      :uploader="uploaderMock"
    />
<!--
    <s-file-upload
      color="teal-10"
      dropColor="teal-3"
      progressColor="teal-10"
      modal="minimized"
      buttonModal="Upload de Arquivos para o Modal"
      float-label="Cadastro de Arquivos - Modal"
      accept="image/jpeg, image/jpg, image/png, image/gif"
      multiple
      :uploader="uploaderMock"
    /> -->

  </div>
</template>

<script>
import SFileUpload from './UploaderBody.vue'

const uploaderMock = {
  save(file) {
    return {
      on(evt, fn) {
        if (evt === 'progress') {
          listenerFactory(({progress, done}) => {
            if (done) {
              listenerFactory((remoteObj) => {
                fn({progress, done}, remoteObj);
              });
            } else {
              fn({progress, done}, { progress: 0, done: false })
            }
          })
        }
      }
    }
  },

  saveOnlyRemote (file) {
    return {
      on (evt, fn) {
        if (evt === 'progress') {
          listenerFactory(fn)
        }
      }
    }
  }
}

function listenerFactory(fn) {
  let done = false;
  let progress = 0;

  const interval = setInterval(() => {
    if (!done) {
      progress += getRandomInt(1, 10);

      if (progress >= 100) {
        progress = 100;
        done = true;
      }

      fn({ progress, done, cancel });
      return;
    }
    cancel();
  }, 500);

  function cancel() {
    clearInterval(interval);
  }
}

function getRandomInt(min, max) {
  min = Math.ceil(min)
  max = Math.floor(max)
  return Math.floor(Math.random() * (max - min)) + min
}

export default {
  components: {
    SFileUpload
  },

  data () {
    return {
      uploaderMock
    };
  }
}
</script>

<style lang="stylus">
  .fill-height {
    min-height: 100%;
    display: flex;
    flex-direction: column;
  }
</style>
