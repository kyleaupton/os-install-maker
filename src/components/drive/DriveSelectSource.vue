<template>
  <!-- If file is chosen -->
  <template v-if="file">
    <div class="iso-wrap iso-preview">
      <div class="iso-preview-file">
        <font-awesome-icon
          class="iso-chooser-icon"
          :icon="['fab', 'windows']"
        />
        <div class="iso-preview-path">{{ fileName }}</div>
        <font-awesome-icon
          class="iso-preview-close"
          :icon="['fas', 'xmark']"
          @click="resetFile"
        />
      </div>
    </div>
  </template>

  <!-- If no file is chosen -->
  <template v-else-if="!loading">
    <!-- <Drop @drop="handleDrop">
      <div class="iso-wrap iso-chooser">
        <font-awesome-icon
          class="iso-chooser-icon"
          style="font-size: 48px"
          :icon="['fab', 'windows']"
        />
        <div class="iso-chooser-title">Drop Windows install here</div>
        <div>or</div>
        <button @click="handleOpenFile">Choose File</button>
      </div>
    </Drop> -->
    got here
  </template>
</template>

<script lang="ts">
import { defineComponent, PropType } from 'vue';
import Drive from '@/api/Drive';
import { getErrorMessage } from '@/utils/error';
import { t_file } from '@/types/iso';
// import Drop from '@/components/Drop.vue';

export default defineComponent({
  name: 'IsoSelector',

  components: {
    // Drop,
  },

  props: {
    drive: {
      type: Object as PropType<Drive>,
      required: true,
    },
  },

  emits: ['fileChange'],

  data() {
    return {
      file: undefined as t_file | undefined,
      error: '',
      loading: true,
    };
  },

  computed: {
    fileName() {
      return this.file ? this.file.path.split('/').pop() : '';
    },
  },

  watch: {
    file() {
      this.$emit('fileChange', this.file);
    },
  },

  created() {
    this.getSuggestionList();
  },

  methods: {
    async getSuggestionList() {
      let path = '';

      if (this.drive.os === 'Windows' || this.drive.os === 'Linux') {
        const homeDir = await window.api.ipc.invoke('/utils/os/home');
        path = `${homeDir}/Downloads`;
      } else if (this.drive.os === 'macOS') {
        path = '/Applications';
      }

      const files = await window.api.ipc.invoke('/utils/fs/readdir', { path });

      console.log(files);

      this.loading = false;
    },

    handleDrop(e: DragEvent) {
      if (e.dataTransfer) {
        try {
          const { files } = e.dataTransfer;

          /**
           * EDGE CASE: More than one file
           */
          if (files.length > 1) {
            throw Error('Only one file is allowed');
          }

          const file = files[0];

          /**
           * EDGE CASE: Something that is not a .iso file
           */
          // @ts-ignore - File types are screwed up
          if (!file.path.split('/').pop()?.endsWith('.iso')) {
            throw Error('Only .iso files are allowed');
          }

          this.file = {
            name: file.name,
            // @ts-ignore - File types are screwed up
            path: file.path,
            size: file.size,
          };
        } catch (e) {
          console.error(e);
          this.error = getErrorMessage(e);
        }
      }
    },

    async handleOpenFile() {
      const path = (await window.api.showOpenIsoDialog()).filePaths[0];
      this.file = await window.api.getFileFromPath(path);
    },

    resetFile() {
      this.file = undefined;
    },
  },
});
</script>

<style scoped>
.iso-wrap {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 16px;
}

.iso-preview-file {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 16px;
  text-align: center;
  margin-top: 32px;
}

.iso-preview-path {
  font-weight: 600;
}

.iso-chooser {
  gap: 16px;
  padding: 16px;
  border: 2px dashed var(--text-1);
  border-radius: 4px;
}

.iso-chooser-icon {
  font-size: 28px;
}

.iso-chooser-title {
  font-size: 18px;
  font-weight: 600;
}

.iso-preview-close {
  cursor: pointer;
}

/* .iso-preview-close:hover {
  background-color: ;
} */
</style>