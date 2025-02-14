<template>

  <div
    ref="sidePanel"
    :class="{ 'is-rtl': isRtl, 'is-mobile': isMobile }"
    @keyup.esc="closePanel"
  >
    <transition name="side-panel">
      <div
        class="side-panel"
        :style="sidePanelStyles"
      >

        <!-- Fixed header with optional close button -->
        <div v-if="$slots.header" ref="fixedHeader" class="fixed-header" :style="fixedHeaderStyles">

          <div class="header-content" tabindex="0">
            <slot name="header">
            </slot>
          </div>

        </div>

        <KIconButton
          v-if="!closeButtonHidden"
          icon="close"
          class="close-button"
          :style="closeButtonStyles"
          :ariaLabel="coreString('closeAction')"
          :tooltip="coreString('closeAction')"
          @click="closePanel"
        />

        <!-- Default slot for inserting content which will scroll on overflow -->
        <div class="side-panel-content" :style="contentStyles">
          <slot></slot>
        </div>

      </div>
    </transition>

    <Backdrop
      :transitions="true"
      class="backdrop"
      @click="closePanel"
    />
  </div>

</template>


<script>

  import Backdrop from 'kolibri.coreVue.components.Backdrop';
  import commonCoreStrings from 'kolibri.coreVue.mixins.commonCoreStrings';
  import responsiveWindowMixin from 'kolibri.coreVue.mixins.responsiveWindowMixin';

  export default {
    name: 'FullScreenSidePanel',
    components: {
      Backdrop,
    },
    mixins: [responsiveWindowMixin, commonCoreStrings],
    props: {
      /* Hides the (X) icon button to close the side panel. In this case, clicking off of the
         panel or hitting the ESC keys are the only way to close the panel */
      closeButtonHidden: {
        type: Boolean,
        default: false,
      },
      /* Optionally override the default width of the side panel with valid CSS value */
      sidePanelWidth: {
        type: String,
        required: false,
        default: '436px',
      },
      /* Which side of the screen should the panel be fixed? Reverses the value when isRtl */
      alignment: {
        type: String,
        required: true,
        validator(value) {
          return ['right', 'left'].includes(value);
        },
      },
    },
    data() {
      return {
        /* Will be calculated in mounted() as it will get the height of the fixedHeader then */
        fixedHeaderHeight: null,
      };
    },
    computed: {
      isMobile() {
        return this.windowBreakpoint == 0;
      },
      /* Returns an object with properties left or right set to the appropriate value
         depending on isRtl and this.alignment */
      rtlAlignment() {
        if (this.isRtl && this.alignment === 'left') {
          return 'right';
        } else if (this.isRtl && this.alignment === 'right') {
          return 'left';
        } else {
          return this.alignment;
        }
      },
      /* Returns an object with this.rtlAlignment set to 0 */
      langDirStyles() {
        return {
          [this.rtlAlignment]: 0,
        };
      },
      responsiveWidth() {
        return this.isMobile ? '100vw' : this.sidePanelWidth;
      },
      /** Styling Properties */
      fixedHeaderStyles() {
        return {
          ...this.langDirStyles,
          width: this.responsiveWidth,
          position: 'fixed',
          top: 0,
          backgroundColor: this.$themeTokens.surface,
          'border-bottom': `1px solid ${this.$themePalette.grey.v_500}`,
          padding: '24px 32px',
          // Header border stays over content with this, but under any tooltips
          'z-index': 16,
          // Ensure the content doesn't overlap the close button when present, accounts for RTL
          [`padding-${this.rtlAlignment}`]: this.closeButtonHidden ? 0 : '80px',
        };
      },
      sidePanelStyles() {
        return {
          ...this.langDirStyles,
          width: this.responsiveWidth,
          top: 0,
          position: 'fixed',
          color: this.$themeTokens.text,
          backgroundColor: this.$themeTokens.surface,
          'z-index': 12,
        };
      },
      contentStyles() {
        return {
          'margin-top': this.fixedHeaderHeight,
          padding: '24px 32px 16px',
          'overflow-y': 'scroll',
          height: `calc((100vh - ${this.fixedHeaderHeight}))`,
        };
      },
      closeButtonStyles() {
        return {
          top: `calc((${this.fixedHeaderHeight} - 40px) / 2)`,
        };
      },
    },
    /* this is the easiest way I could think to avoid having dual scroll bars */
    mounted() {
      const htmlTag = window.document.getElementsByTagName('html')[0];
      htmlTag.style['overflow-y'] = 'hidden';

      // Gets the height of the fixed header - adds 40 to account for padding
      this.fixedHeaderHeight = this.$refs.fixedHeader.clientHeight + 'px';

      // Ensures user starts at top header with keyboard focus
      this.$refs.fixedHeader.focus();
    },
    beforeDestroy() {
      const htmlTag = window.document.getElementsByTagName('html')[0];
      htmlTag.style['overflow-y'] = 'auto';
    },
    methods: {
      closePanel() {
        this.$emit('closePanel');
      },
    },
    $trs: {
      /* eslint-disable kolibri/vue-no-unused-translations */
      topicHeader: {
        message: 'Also in this folder',
        context: 'Title of the panel with all topic contents. ',
      },
    },
  };

</script>


<style lang="scss" scoped>

  @import '~kolibri-design-system/lib/styles/definitions';

  .header-content {
    width: 100%;
  }

  .close-button {
    position: fixed;
    right: 32px;
    z-index: 24;
  }

  /** Need to be sure a KDropdownMenu shows up on the Side Panel */
  /deep/ .tippy-popper {
    z-index: 24;
  }

</style>
