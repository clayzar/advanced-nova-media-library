<template>
  <gallery-item class="gallery-item-image">
    <div class="gallery-item-info p-3">
      <a v-if="downloadUrl" class="icon download" :href="downloadUrl" title="Download">
        <icon type="download" view-box="0 0 20 22" width="16" height="16"/>
      </a>
      <a v-if="removable" class="icon delete" href="#" @click.prevent="$emit('remove')" title="Remove">
        <icon type="delete" view-box="0 0 20 20" width="16" height="16"/>
      </a>
      <a v-if="isCustomPropertiesEditable" class="icon edit" href="#" @click.prevent="$emit('editCustomProperties')" title="Edit custom properties">
        <icon type="edit" view-box="0 0 20 20" width="16" height="16"/>
      </a>
      <a class="preview" href="#" @click.prevent="showPreview">
        <icon type="search" view-box="0 0 20 20" width="30" height="30"/>
      </a>
      <a v-if="croppable" class="icon crop" href="#" @click.prevent="$emit('crop-start', image)">
        <scissors-icon brand="var(--info)" view-box="0 0 20 20" width="16" height="16"/>
      </a>
    </div>
    <img :src="src" :alt="image.name" class="gallery-image">
  </gallery-item>
</template>

<script>
  import ScissorsIcon from './icons/Scissors';
  import GalleryItem from './GalleryItem';

  export default {
    components: {
      ScissorsIcon,
      GalleryItem,
    },
    props: ['image', 'field', 'removable', 'editable', 'isCustomPropertiesEditable'],
    data: () => ({
      acceptedMimeTypes: ['image/jpg', 'image/jpeg', 'image/png'],
      src: "data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==",
    }),
    computed: {
      downloadUrl() {
        return this.image.id ? `/nova-vendor/ebess/advanced-nova-media-library/download/${this.image.id}` : null;
      },
      croppable() {
        return this.editable &&
          this.field.croppable &&
          this.acceptedMimeTypes.includes(this.image.mime_type || this.image.file.type);
      },
    },
    watch: {
      image: {
        handler: 'getImage',
        immediate: true
      }
    },
    methods: {
      showPreview() {
        const blobUrl = this.image.file ? URL.createObjectURL(this.image.file) : this.image.full_urls.default;
        window.open(blobUrl, '_blank');
      },
      getImage() {
        // Return desired image conversion on view if it exists
        let conversionOnView = this.field.conversionOnView;

        if (this.image.id && conversionOnView && this.image.full_urls[conversionOnView]) {
          this.src = this.image.full_urls[conversionOnView];
          return;
        }

        // Return thumbnail if conversion exists
        let thumbnail = this.field.thumbnail;

        if (this.image.id && thumbnail && this.image.full_urls[thumbnail]) {
          this.src = this.image.full_urls[thumbnail];
          return;
        }

        if (this.isVideo(this.image.full_urls.default)) {
          //Seconds to seek to, to get thumbnail of video
          let seconds = 1;  //TODO get this from the field instead of hardcoding it here
          this.getVideoThumbnail(this.image.full_urls.default, seconds);
          return;
        }

        this.src = this.image.full_urls.default;
      },
      getVideoThumbnail(path, secs = 0) {
        const video = document.createElement('video');
        video.onloadedmetadata = () => {
          video.currentTime = Math.min(Math.max(0, (secs < 0 ? video.duration : 0) + secs), video.duration);
        };
        video.onseeked = (e) => {
          const canvas = document.createElement('canvas');
          canvas.height = video.videoHeight;
          canvas.width = video.videoWidth;
          const ctx = canvas.getContext('2d');
          ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
          this.src = canvas.toDataURL();
        };
        video.src = path;
      },
      isVideo(mediaPath) {
        if (mediaPath.startsWith("data:video"))
          return true;
        //TODO better video detection
        const supportedExtensions = [".mp4", ".ogv", ".webm"];
        return supportedExtensions.some((suffix) => {
          return mediaPath.endsWith(suffix)
        });
      }
    },
  };
</script>

<style lang="scss">
  $bg-color: #e8f5fb;
  $item-max-size: 150px;
  $border-radius: 10px;

  .gallery {
    .gallery-item-image.gallery-item {
      width: $item-max-size;
      height: $item-max-size;

      &:hover .gallery-item-info {
        display: flex;
      }

      .gallery-item-info {
        display: none;
        align-items: center;
        justify-content: center;
        flex-direction: column;
        background-color: transparentize($bg-color, .2);
        border-radius: $border-radius;
        position: absolute;
        z-index: 10;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;

        .preview {
          color: var(--black);
        }

        .delete {
          right: 10px;
          color: var(--danger);
        }

        .crop {
          left: 10px;
          top: auto;
          bottom: 10px;
        }
      }

      .gallery-image {
        object-fit: contain;
        display: block;
        max-height: 100%;
        border-radius: $border-radius;
      }
    }

    .icon {
      cursor: pointer;
      position: absolute;
      top: 10px;
      color: var(--info);
    }

    .edit {
      right: 30px;
    }

    .download {
      left: 10px;
    }
  }
</style>
