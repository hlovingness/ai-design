<template>
    <a-layout-footer class="dea-footer-page">
        <div class="bg-white page-box not-select">
            <a-space>
                <div class="page-view " v-for="(item,index) in workspacesData"
                     @click="onSelect(item)"
                     @contextmenu.stop="openContextMenu($event,item)"
                     :class="{'page-selected':workspaces.getCurrentId() === item.id}"
                     :key="index">
                    <a-avatar class="page-ava" :size="30" shape="square">{{ index + 1 }}</a-avatar>
                </div>

                <div class="page-add page-view" @click="addOnClick">
                    <icon-plus size="20"/>
                </div>
            </a-space>
        </div>
    </a-layout-footer>
    <div style="background: #f1f2f4">
        <div class="flex justify-center flex-items-center p-b-5px" style="color: var(--color-text-2)">
            <span>本网站由</span>
            <a-link :hoverable="false" target="_blank"
                    href="https://www.upyun.com/?utm_source=lianmeng&utm_medium=referral">
                <img class="h-28px m-l-4px m-r-4px" src="@/assets/images/又拍云_logo5.png">
            </a-link>
            <span>提供CDN加速/云存储服务</span>
        </div>
    </div>
</template>
<script setup lang="ts">
import {useEditor} from "@/views/Editor/app";
import ContextMenu from '@/components/contextMenu'

const {canvas, workspaces, event} = useEditor()
import {IWorkspace} from '@/views/Editor/core/workspaces/workspacesService'

const pages = computed(() => {
    return canvas.getPages()
})
const addOnClick = () => {
    workspaces.setCurrentId(workspaces.add(`${(pages.value.size + 1)}`))
    canvas.zoomToFit()
}
const workspacesData = ref<IWorkspace[]>([])
const updateWorkspaces = () => {
    workspacesData.value = workspaces.all().map((workspace) => ({
        id: workspace.id,
        name: workspace.name,
        cover: workspace.cover
    }));
}

updateWorkspaces()

event.on('workspaceChangeAfter', updateWorkspaces)
event.on('workspaceAddAfter', updateWorkspaces)
event.on('workspaceRemoveAfter', updateWorkspaces)

onUnmounted(() => {
    event.off('workspaceChangeAfter', updateWorkspaces)
    event.off('workspaceAddAfter', updateWorkspaces)
    event.off('workspaceRemoveAfter', updateWorkspaces)
})

const onSelect = (item: IWorkspace) => {
    workspaces.setCurrentId(item.id.toString())
}

const openContextMenu = (e: MouseEvent, node: any) => {
    e.preventDefault()
    ContextMenu.showContextMenu({
        x: e.clientX,
        y: e.clientY,
        preserveIconWidth: false,
        items: [
            {
                label: '复制',
                onClick: async () => {
                    if (!node.id) return
                    const workspace = workspaces.get(node.id.toString())
                    if (!workspace) return
                    const id = workspaces.add(`${(pages.value.size + 1)}`)
                    workspaces.setCurrentId(id)
                    // 循序不能变， getPageJSON必须在setCurrentId之后执行，否则要复制的页面数据可能还未保存
                    const json = canvas.getPageJSON(node.id)
                    canvas.reLoadFromJSON(json)
                },
            },
            {
                label: '删除',
                disabled: workspaces.size() <= 1 || node.id === workspaces.getCurrentId(),
                onClick: () => {
                    if (!node.id) return
                    workspaces.remove(node.id.toString())
                },
                // divided: true,
            },
            // {
            //     label: '重命名',
            //     onClick: () => {
            //
            //     },
            // },
        ],
    })
}
</script>

<style lang="less" scoped>
@import "../../styles/layouts";

.dea-footer-page {
  background-color: #f1f2f4;
  padding: 10px 20px 10px 20px;
  height: @footerBoxHeight;
  overflow: auto;

  .page-box {
    padding: 0 5px;
    height: 100%;
    align-items: center;
    display: flex;
  }

  .page-view {
    cursor: pointer;
    border: 1px solid var(--color-neutral-4);
    border-radius: 5px;
    padding: 5px;
    height: calc(@footerBoxHeight - 30px);
    align-items: center;
    align-content: center;
    display: flex;
  }

  .page-selected {
    border-color: rgb(var(--primary-6));
  }

  .page-add {
    height: calc(@footerBoxHeight - 30px);
    width: calc(@footerBoxHeight - 30px);
    text-align: center;
    align-items: center;
    display: flex;
    align-content: center;
    justify-content: center;
  }

  .page-add:hover {
    color: rgb(var(--primary-6));
    border-color: rgb(var(--primary-6));
    cursor: pointer;
  }

  .page-ava {
    //background-color: #3370ff;
  }
}
</style>
