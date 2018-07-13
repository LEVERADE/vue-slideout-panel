<template src="./template.html">
</template>

<style lang="less" src="./styles.less"></style>

<script>
import eventBus from '../../eventBus';

const vm = {
  name: 'slideout-panel',
  data() {
    return {
      visible: false,
      panelsVisible: false,
      panels: [],
      zIndexBg: 99
    };
  },
  methods: {
    onPanelFullyClosed(el) {
      let lastPanel = this.panels[this.panels.length - 1];
      if (lastPanel !== undefined) {
        lastPanel.styles['z-index'] += 1;
        this.zIndexBg = lastPanel.styles['z-index'] - 1;
      }
    },
    getPanelClasses(panel) {
      const panelClasses = {};

      if (panel.openOn === 'left') {
        panelClasses['open-on-left'] = true;
      } else {
        panelClasses['open-on-right'] = true;
      }

      if (panel.cssClass) panelClasses[panel.cssClass] = true;

      return panelClasses;
    },
    onCloseComponent(data) {
      this.closeCurrentPanel(data);
    },
    closeCurrentPanel(data) {
      const currentPanel = this.panels[this.panels.length - 1];

      if (currentPanel !== undefined) {

        eventBus.$emit(`hideSlideOutPanel-${currentPanel.id}`, {
          id: currentPanel.id,
          data
        });

        const index = this.panels.indexOf(currentPanel);

        this.panels.splice(index, 1);

        if (!this.panels || this.panels.length === 0) {
          this.onLastPanelDestroyed();
        }

      }
    },
    onShowSlideOutPanel(panel) {
      panel.styles = {};

      let zIndex = 100;
      if (panel.zIndex !== undefined) {
        zIndex = Number(panel.zIndex);
      }

      this.panels.forEach((item, index) => {
          item.styles['z-index'] = zIndex + index;
      });

      panel.styles['z-index'] = zIndex + this.panels.length + 1;
      this.zIndexBg = panel.styles['z-index'] - 1;

      if (panel.width !== 'auto') {
          if (panel.width === undefined) panel.styles.width = '900px';
          else if (isNaN(panel.width)) panel.styles.width = panel.width;
          else panel.styles.width = `${panel.width}px`;
      }

      this.panels.push(panel);

      if (!this.panels || this.panels.length === 1) {
        this.onFirstPanelCreated();
      }
    },
    onFirstPanelCreated() {
      this.visible = true;

      setTimeout(() => {
        this.panelsVisible = true;
      }, 300);

      document.addEventListener('keydown', this.onEscapeKeypress);

      document.body.className = (document.body.className + ' slideout-panel-open').trim();
    },
    onLastPanelDestroyed() {
      this.panelsVisible = false;

      setTimeout(() => {
        this.visible = false;
      }, 300);

      document.removeEventListener('keydown', this.onEscapeKeypress);

      document.body.className = document.body.className.replace('slideout-panel-open', '').trim();
    },
    onBgClicked() {
      console.log('slideout-panel: Background clicked');

      this.closeCurrentPanel();
    },
    onEscapeKeypress(e) {
      if (e.keyCode === 27) {
        console.log('slideout-panel: Escape pressed');

        this.closeCurrentPanel();
      }
    }
  },
  created() {
    eventBus.$on('showSlideOutPanel', this.onShowSlideOutPanel);
    eventBus.$on('hideSlideOutPanel', this.closeCurrentPanel);
  },
  destroyed() {
    eventBus.$off('showSlideOutPanel', this.onShowSlideOutPanel);
    eventBus.$off('hideSlideOutPanel', this.closeCurrentPanel);
  }
};

export default vm;
</script>
