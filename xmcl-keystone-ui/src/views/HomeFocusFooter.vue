<template>
  <div class="w-full">
    <div class="relative mx-6 flex-grow flex gap-6 items-end">
      <!-- <v-card> -->
        <!-- <v-img>

        </v-img>
        <v-card-text>
          123
        </v-card-text> -->
        <!-- <HomeModCard class="min-h-60" :row="1" :row-count="8" /> -->
      <!-- </v-card> -->
      <!-- <v-btn
        v-shared-tooltip="_ => t('baseSetting.title')"
        text
        icon
        to="/base-setting"
        :loading="refreshing"
      >
        <v-icon>
          tune
        </v-icon>
      </v-btn>
      <v-btn
        v-shared-tooltip="_ => t('instance.showInstance')"
        text
        icon
        @click="showInstanceFolder"
      >
        <v-icon>
          folder
        </v-icon>
      </v-btn>

      <v-btn
        v-shared-tooltip="_ => t('logsCrashes.title')"
        text
        icon
        @click="showLogDialog"
      >
        <v-icon>
          subtitles
        </v-icon>
      </v-btn>
      <v-btn
        v-shared-tooltip="_ => t('modpack.export')"
        text
        icon
        :loading="refreshing"
        @click="showExport"
      >
        <v-icon>
          share
        </v-icon>
      </v-btn> -->
      <v-spacer />
      <HomeHeaderInstallStatus
        v-if="status === 1 || status === 3"
        class="mr-2"
        :name="taskName"
        :total="total"
        :progress="progress"
      />
      <HomeLaunchButtonStatus
        v-else
        :active="active"
      />
      <HomeLaunchButton
        class="ml-4"
        :status="status"
        @pause="pause"
        @resume="resume"
        @mouseenter="active = true"
        @mouseleave="active = false"
      />
    </div>
  </div>
</template>
<script lang="ts" setup>
import { useService } from '@/composables'
import { useDialog } from '@/composables/dialog'
import { kInstance } from '@/composables/instance'
import { AppExportDialogKey } from '@/composables/instanceExport'
import { kLaunchTask } from '@/composables/launchTask'
import { injection } from '@/util/inject'
import { BaseServiceKey } from '@xmcl/runtime-api'
import HomeLaunchButton from './HomeLaunchButton.vue'
import HomeHeaderInstallStatus from './HomeHeaderInstallStatus.vue'
import HomeLaunchButtonStatus from './HomeLaunchButtonStatus.vue'

const active = ref(false)
const { path, refreshing } = injection(kInstance)
const { total, progress, status, name: taskName, pause, resume } = injection(kLaunchTask)
const { openDirectory } = useService(BaseServiceKey)
const { show: showExport } = useDialog(AppExportDialogKey)
const { show: showLogDialog } = useDialog('log')
const { t } = useI18n()

function showInstanceFolder() {
  openDirectory(path.value)
}
</script>
