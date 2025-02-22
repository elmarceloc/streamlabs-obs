<template>
  <modal-layout :doneHandler="handleSelect">
    <div
      slot="content"
      class="container"
      @dragenter.prevent="onDragEnter"
      @dragover.prevent="onDragOver"
      @drop.prevent="handleFileDrop($event)"
    >
      <input
        type="file"
        id="media-gallery-input"
        @change="handleUploadClick($event)"
        accept=".webm,.gif,.jpg,.png,.mp3,.ogg,.wav,.svg,.eps,.ai,.psd"
        multiple="multiple"
        style="display: none;"
      />
      <div class="flex" style="height: 100%;">
        <div class="left-panel">
          <div class="dropzone" @click="openFilePicker">
            <i class="icon-cloud-backup"></i>{{ $t('Drag & Drop Upload') }}
          </div>
          <ul v-for="cat in ['uploads', 'stock']" :key="cat" class="nav-list">
            <div>
              <div class="list__title">
                {{ cat === 'stock' ? $t('Stock Files') : $t('My Uploads') }}
              </div>
              <li
                class="list__item"
                :class="{ active: type === null && cat === category }"
                @click="handleTypeFilter(null, cat)"
              >
                <i class="fa fa-file"></i>{{ $t('All Files') }}
              </li>
              <li
                class="list__item"
                :class="{ active: type === 'image' && cat === category }"
                @click="handleTypeFilter('image', cat)"
              >
                <i class="icon-image"></i>{{ $t('Images') }}
              </li>
              <li
                class="list__item"
                :class="{ active: type === 'audio' && cat === category }"
                @click="handleTypeFilter('audio', cat)"
              >
                <i class="icon-music"></i>{{ $t('Sounds') }}
              </li>
            </div>
          </ul>
          <div>
            <div>{{ totalUsageLabel }} / {{ maxUsageLabel }}</div>
            <div class="progress-slider radius">
              <div
                :style="'width: ' + usagePct * 100 + '%'"
                class="progress-slider__fill radius"
              ></div>
            </div>
          </div>
        </div>
        <div class="right-panel">
          <h4>{{ title }}</h4>
          <div class="toolbar">
            <i class="icon-cloud-backup" @click="openFilePicker"></i>
            <i
              class="icon-trash"
              :class="{ disabled: !selectedFile || (selectedFile && selectedFile.isStock) }"
              @click="handleDelete"
            ></i>
            <i
              class="fa fa-download"
              :class="[!selectedFile ? 'disabled' : '']"
              @click="handleDownload"
            ></i>
          </div>
          <div
            v-if="dragOver"
            @dragover.prevent="onDragOver"
            @dragleave.prevent="onDragLeave"
            class="drag-overlay radius"
          ></div>
          <div v-if="busy" class="busy-overlay"></div>
          <scrollable v-if="files.length" className="uploads-manager__list">
            <li
              v-for="file in files"
              :key="file.href"
              :class="[selectedFile && selectedFile.href === file.href ? 'selected' : '']"
              class="uploads-manager__item radius"
              @click.prevent="selectFile(file)"
              @dblclick.prevent="selectFile(file, true)"
            >
              <div v-if="file.type === 'image' && /\.webm$/.test(file.href)">
                <video
                  autoplay
                  muted
                  loop
                  :src="file.href"
                  style="height: 100%; width: 100%;"
                ></video>
              </div>
              <div
                v-if="file.type == 'image' && !/\.webm$/.test(file.href)"
                class="image-preview"
                :style="'background-image: url(' + file.href + ')'"
              ></div>
              <div v-if="file.type == 'audio'" style="height: 132px;">
                <i
                  class="icon-music"
                  style="line-height: 132px; font-size: 28px; text-align: center; display: block;"
                />
              </div>
              <i
                v-if="!file.prime"
                class="icon-copy"
                v-tooltip.left="$t('Copy URL')"
                @click="handleCopy(file.href)"
              />
              <div class="upload__footer" :class="[file.type === 'image' ? 'image' : '']">
                <div class="upload__size">{{ file.size ? formatBytes(file.size) : ' ' }}</div>
                <div class="upload__title">{{ file.filename }}</div>
                <div v-if="file.prime" class="upload__prime">
                  {{ $t('Prime') }}
                  <i class="icon-prime" />
                </div>
              </div>
            </li>
          </scrollable>

          <div v-if="!files.length" class="empty-box">
            <div>{{ noFilesCopy }}</div>
            <div>
              <button @click="openFilePicker" class="button">{{ noFilesBtn }}</button>
              <button @click="handleBrowseGalleryClick" class="button">
                {{ $t('Browse the Gallery') }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </modal-layout>
</template>

<script lang="ts" src="./MediaGallery.vue.ts"></script>

<style lang="less">
.uploads-manager__list {
  .os-content {
    display: flex;
    margin: 10px 0 0;
    flex-wrap: wrap;
  }
}
</style>

<style lang="less" scoped>
@import '../../styles/index';

.container {
  display: block;
  position: relative;
  top: 0;
  left: 0;
  padding: 30px;
  width: 100%;
  height: 100%;
  font-family: Roboto, sans-serif;
}

.header {
  font-size: 16px;
  letter-spacing: 0;
  margin-bottom: 30px;
  color: var(--title);
}

.left-panel {
  width: 180px;
}

.dropzone {
  border: 2px dashed var(--teal);
  color: var(--teal);
  text-align: center;
  padding: 10px 20px;
  font-size: 12px;
  margin-bottom: 20px;

  i {
    display: block;
    font-size: 24px;
  }

  &:hover {
    background-color: var(--teal-semi);
    color: var(--teal);
    border-color: var(--teal);
    cursor: pointer;
  }
}

.nav-list {
  list-style: none;
  margin-left: 0;
}

.list__title {
  padding: 0;
  .weight(@bold);
}

.list__item {
  padding: 3px 8px;
  color: var(--paragraph);
  .weight(@medium);

  i {
    color: var(--icon);
    padding-right: 6px;
  }

  &:hover {
    cursor: pointer;
  }
}

.list__item.active {
  background-color: var(--button);
}

.progress-slider {
  position: relative;
  width: 100%;
  height: 6px;
  margin-bottom: 0;
  margin-top: 6px;
  background-color: var(--border);
}

.progress-slider__fill {
  background-color: var(--paragraph);
  height: 6px;
  position: absolute;
  top: 0;
  left: 0;
}

.right-panel {
  width: 100%;
  height: 100%;
  padding-left: 30px;
  overflow: hidden;
}

.toolbar {
  .padding();
  .radius();

  border: 1px solid var(--teal-semi);
  background: var(--section);
  width: 100%;

  i {
    font-size: 20px;
    color: var(--icon);
    margin-right: 8px;

    &:hover {
      color: var(--teal);
      cursor: pointer;
    }
  }

  i.disabled {
    color: var(--solid-input);

    &:hover {
      cursor: default;
    }
  }
}

.empty-box {
  text-align: center;
  margin-top: 160px;
}

.submit-container {
  display: flex;
  justify-content: flex-end;
  position: relative;
  left: 0;
  right: 0;
  padding-right: 30px;

  button {
    margin: 6px;
  }
}

.drag-overlay {
  border: 1px solid var(--info);
  background: rgba(255, 200, 0, 0.15);
  z-index: 100000;
  box-sizing: border-box;
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  width: 100%;
}

.busy-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border: 1px solid var(--section);
  background: var(--shadow);
  z-index: 99999;
  .radius;
}

.uploads-manager__list {
  padding: 0;
  list-style: none;
  height: calc(100% - 64px);
  width: 100%;
  padding-bottom: 30px;
}

.uploads-manager__item {
  border: 1px solid var(--border);
  width: 23%;
  margin: 0 14px 14px 0;
  height: 170px;
  cursor: default;
  position: relative;
  overflow: hidden;

  &:hover {
    border-color: var(--teal);
    background-color: var(--teal-semi);
    cursor: pointer;
  }

  &.selected {
    border-color: var(--teal);
    background-color: var(--teal-semi);
  }

  .icon-copy {
    position: absolute;
    top: 8px;
    right: 8px;
    display: block;

    &:hover {
      color: var(--title);
    }
  }
}

.image-preview {
  height: 132px;
  background-size: contain;
  background-position: center;
  background-repeat: no-repeat;
}

.upload__size {
  color: var(--white);
  text-transform: uppercase;
  font-size: 12px;
}

.upload__prime {
  color: var(--prime);
}

.upload__footer {
  position: absolute;
  bottom: 0;
  display: inline-block;
  width: 100%;
  padding: 24px 10px 10px 10px;
  border-bottom-left-radius: 3px;
  border-bottom-right-radius: 3px;

  &.image {
    background: linear-gradient(
      rgba(55, 71, 79, 0),
      rgba(55, 71, 79, 0.3),
      rgba(55, 71, 79, 0.6),
      rgba(55, 71, 79, 0.9)
    );

    .upload__title {
      color: var(--white);
    }

    .upload__size {
      color: var(--icon);
    }
  }
}
</style>
