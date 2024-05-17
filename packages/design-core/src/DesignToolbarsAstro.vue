<template>
  <div class="tiny-engine-toolbar">
    <div class="toolbar-left">
      <component :is="item.component" v-for="item in leftBar" :key="item.id"></component>
    </div>
    <div class="toolbar-center">
      <template v-for="item in centerBar" :key="item.id">
        <component
          :is="item.component"
          :icon="item.icon"
          :undoIcon="item.undoIcon"
          :redoIcon="item.undoIcon"
        ></component>
        <span v-if="item?.splitLines">|</span>
      </template>
    </div>
    <div class="toolbar-right">
      <component :is="item.component" v-for="item in rightBar" :key="item.id"></component>
    </div>
  </div>
  <div class="progress">
    <progress-bar v-if="state.showDeployBlock"></progress-bar>
  </div>
</template>

<script>
import { reactive, nextTick } from 'vue'
import addons from '@opentiny/tiny-engine-app-addons'
import { useLayout } from '@opentiny/tiny-engine-controller'
import { ProgressBar } from '@opentiny/tiny-engine-common'
export default {
  components: {
    ProgressBar
  },
  setup() {
    const leftBar = []
    const rightBar = []
    const centerBar = []
    const state = reactive({
      showDeployBlock: false
    })
    const astroToolbarsMap = [
      {
        id: 'breadcrumb',
        align: 'left'
      },
      {
        id: 'redoundo',
        undoIcon: 'undo-astro',
        redoIcon: 'redo-astro',
        align: 'center'
      },
      {
        id: 'save',
        icon: 'save-astro',
        align: 'center'
      },
      {
        id: 'refresh',
        align: 'center',
        icon: 'refresh-astro',
        splitLines: true
      },
      {
        id: 'setting',
        icon: 'setting-astro',
        align: 'center'
      },
      {
        id: 'lang',
        align: 'center',
        splitLines: true
      },
      {
        id: 'media',
        align: 'center'
      },
      {
        id: 'user',
        align: 'right'
      },
      {
        id: 'preview',
        align: 'right'
      },
      {
        id: 'genarate-vue',
        align: 'right'
      }
    ]
    const astroToolbars = astroToolbarsMap.map((item) => {
      const toolbarItem = addons.toolbars.find((toolbar) => toolbar.id === item.id)
      return toolbarItem
        ? {
            ...toolbarItem,
            align: item.align,
            splitLines: !!item?.splitLines,
            icon: item?.icon ?? '',
            undoIcon: item?.undoIcon ?? '',
            redoIcon: item?.redoIcon ?? ''
          }
        : item
    })
    astroToolbars.forEach((item) => {
      if (item.align === 'right') {
        rightBar.push(item)
      } else if (item.align === 'center') {
        centerBar.push(item)
      } else {
        leftBar.push(item)
      }
      if (item.id === 'lock') {
        useLayout().registerPluginApi({ Lock: item.api })
      }
      if (item.id === 'save') {
        useLayout().registerPluginApi({ save: item.api })
      }
    })
    nextTick(() => {
      state.showDeployBlock = true
    })

    return {
      leftBar,
      rightBar,
      centerBar,
      state
    }
  }
}
</script>

<style lang="less" scoped>
.tiny-engine-toolbar {
  user-select: none;
  display: flex;
  flex-shrink: 0;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  height: var(--base-top-panel-height);
  text-align: center;
  background-color: var(--ti-lowcode-common-layout-bg);
  position: relative;
  z-index: 1001;
  border-bottom: 1px solid var(--ti-lowcode-toolbar-border-bottom-color);

  .toolbar-left,
  .toolbar-center,
  .toolbar-right {
    height: 100%;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .toolbar-left {
    margin: 0 1px;
  }
  .toolbar-center {
    height: 32px;
    line-height: 32px;
    padding: 0 10px;
    border: 1px solid var(--ti-lowcode-toolbar-border-color);
    border-radius: 30px;
    span {
      color: var(--ti-lowcode-toolbar-border-color);
    }
    :deep(.svg-icon) {
      font-size: 18px !important;
    }
  }

  .toolbar-center,
  .toolbar-right {
    margin: 0 6px;
    margin-right: 24px;
    column-gap: 6px;
    align-items: center;
    :deep(.icon) {
      width: 28px;
      display: inline-flex;
      justify-content: center;
      align-items: center;
      vertical-align: middle;
      border-radius: 6px;
      position: relative;
      svg {
        cursor: pointer;
        font-size: 22px;
        color: var(--ti-lowcode-toolbar-title-color);
      }
      &:not(.disabled):hover {
        background: var(--ti-lowcode-toolbar-view-active-bg);
      }
      &.active {
        color: var(--ti-lowcode-common-primary-color);
      }
      &.disabled {
        cursor: not-allowed;
      }
    }

    .tiny-locales {
      height: 35px;
      padding: 5px 12px;
      font-size: 12px;
      display: inline-flex;
      justify-content: center;
      align-items: center;
      :deep(span) {
        color: var(--ti-lowcode-toolbar-icon-color);
        opacity: 0.4;
      }

      &:hover {
        :deep(span) {
          opacity: 0.8;
        }
      }
    }
  }
}

@media only screen and (max-width: 1280px) {
  .tiny-engine-toolbar {
    :deep(.top-panel-breadcrumb) {
      width: auto;
    }
  }
}
.progress {
  position: absolute;
  top: var(--base-top-panel-height);
  left: 0;
  width: 100%;
  z-index: 1002;
  :deep(.tiny-progress-bar__innerText) {
    display: none;
  }
}
</style>
