<template>
  <div class="top-panel-breadcrumb">
    <svg-icon name="logo" class="logo-icon"></svg-icon>
    <div class="project-info">
      <tiny-button @click="openSetting">
        <template #default>
          <svg-icon name="file-text-astro" />
          <span>应用名称</span>
          <span> / </span>
          <span>{{ breadcrumbData[1] }}</span>
        </template>
      </tiny-button>
    </div>
  </div>
</template>

<script>
import { Button } from '@opentiny/vue'
import { useBreadcrumb, useLayout, usePage, useCanvas, useNotify, useModal } from '@opentiny/tiny-engine-controller'
import { constants } from '@opentiny/tiny-engine-utils'

const { PAGE_STATUS } = constants
export default {
  components: {
    TinyButton: Button
  },
  setup() {
    const { pageState } = useCanvas()
    const { initCurrentPageData, isChangePageData } = usePage()
    const { PLUGIN_NAME, activePlugin, layoutState, isEmptyPage } = useLayout()
    const { getBreadcrumbData } = useBreadcrumb()
    const { confirm, message } = useModal()
    const breadcrumbData = getBreadcrumbData()

    const openPageAndInit = async (api) => {
      const { currentPage } = pageState
      api.openPageSettingPanel()
      const page = await api.getPageById(currentPage.id)
      initCurrentPageData(page)
    }
    const openPageSetting = () => {
      const { pageStatus } = layoutState

      if (pageStatus.state === PAGE_STATUS.Lock) {
        const username = pageStatus.data?.username || ''
        message({
          message: `您点击的页面被${username}锁定，暂时无法编辑，请联系解锁`,
          status: 'info'
        })
        return
      }

      activePlugin(PLUGIN_NAME.AppManage).then((api) => {
        if (isChangePageData()) {
          confirm({
            title: '提示',
            message: `当前页面尚未保存，是否要继续切换?`,
            exec: () => {
              openPageAndInit(api)
            }
          })
          return
        }
        openPageAndInit(api)
      })
    }

    const openSetting = () => {
      if (isEmptyPage()) {
        useNotify({ type: 'warning', message: '请先创建页面' })

        return
      }

      openPageSetting()
    }
    return {
      breadcrumbData,
      openSetting
    }
  }
}
</script>

<style lang="less" scoped>
.top-panel-breadcrumb {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: auto;
  height: 100%;
  font-size: 14px;
  .logo-icon {
    width: 106px;
    height: 40px;
    margin: 0 16px 0 13px;
  }
  .project-info {
    :deep(.tiny-button) {
      border-color: var(--ti-lowcode-toolbar-border-color);
      font-weight: bold;
    }
    svg {
      font-size: 24px;
    }
  }
}
</style>
