<template>
    <div>
        <div
            v-loading="true"
            element-loading-background="#f5f5f5"
            :element-loading-text="$t('utils.Loading')"
            class="default-main ba-main-loading"
        ></div>
        <div v-if="state.showReload" class="loading-footer">
            <el-button @click="refresh" type="warning">{{ $t('utils.Reload') }}</el-button>
        </div>
    </div>
</template>

<script setup lang="ts">
import { onUnmounted, reactive } from 'vue'
import router from '/@/router/index'
import { useMemberCenter } from '/@/stores/memberCenter'
import { useNavTabs } from '/@/stores/navTabs'
import { isAdminApp } from '/@/utils/common'
import { getFirstRoute, routePush } from '/@/utils/router'
let timer: NodeJS.Timer

const navTabs = useNavTabs()
const memberCenter = useMemberCenter()
const state = reactive({
    maximumWait: 1000 * 6,
    showReload: false,
})

const refresh = () => {
    router.go(0)
}

if (isAdminApp() && navTabs.state.tabsViewRoutes) {
    let firstRoute = getFirstRoute(navTabs.state.tabsViewRoutes)
    if (firstRoute) routePush(firstRoute.name)
} else if (memberCenter.state.viewRoutes) {
    let firstRoute = getFirstRoute(memberCenter.state.viewRoutes)
    if (firstRoute) memberCenter.activateMenu(firstRoute)
}

timer = setTimeout(() => {
    state.showReload = true
}, state.maximumWait)

onUnmounted(() => {
    clearTimeout(timer)
})
</script>

<style scoped lang="scss">
.ba-main-loading {
    height: 300px;
    display: flex;
    align-items: center;
    justify-content: center;
}
.loading-footer {
    display: flex;
    align-items: center;
    justify-content: center;
}
</style>
