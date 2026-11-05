<template>
  <section class="about">
    <SettingCard>
      <div class="pa-4">
        <!-- zenith: Zenith Launcher header and XMCL credit -->
        <div class="d-flex align-center flex-wrap gap-4 mb-6">
          <v-img
            :src="logo"
            alt="Zenith Launcher Logo"
            width="64"
            height="64"
            class="mr-4 rounded-lg flex-grow-0 flex-shrink-0"
          ></v-img>
          <div>
            <a
              class="text-h5 font-weight-bold text-decoration-none"
              href="https://github.com/gitzetazero/zenith-launcher"
              target="_blank"
            >
              Zenith Launcher
            </a>
            <div class="text-caption opacity-80 mt-1" data-testid="zenith-xmcl-credit">
              {{ t('zenith.aboutCredit') }}
              (<a href="https://github.com/Voxelum/x-minecraft-launcher" target="_blank">github.com/Voxelum/x-minecraft-launcher</a>)
            </div>
          </div>
          <v-spacer />
          <div class="d-flex align-center gap-2">
            <v-icon color="primary" size="small">verified</v-icon>
            <div class="text-caption font-weight-medium">{{ t('setting.aboutLicense') }}</div>
          </div>
        </div>
        <div class="text-caption opacity-70 mb-6">
          NOT AN OFFICIAL MINECRAFT PRODUCT. NOT APPROVED BY OR ASSOCIATED WITH MOJANG OR MICROSOFT.
        </div>

        <!-- Debug Info Box -->
        <div class="mb-8">
          <div class="text-subtitle-2 font-weight-bold mb-2 opacity-80">
            {{ t('setting.about') }}
          </div>
          <pre
            class="debug-info-code pa-4 text-caption font-mono"
          ><code>{{ debugInfo }}</code></pre>
        </div>

        <v-divider class="my-6 opacity-20" />

        <!-- Contributors Section -->
        <div class="mb-8">
          <h3 :id="contributorsHeadingId" class="text-subtitle-1 font-weight-bold text-center mb-4">
            {{ t('setting.aboutContributors') }}
          </h3>
          <div
            v-roving-tabindex
            role="group"
            :aria-labelledby="contributorsHeadingId"
            class="d-flex flex-wrap gap-2 justify-center py-2 max-h-48 overflow-y-auto"
          >
            <a
              v-for="c in contributors"
              :key="c.login"
              v-shared-tooltip="() => c.login"
              :href="c.html_url"
              target="_blank"
              class="contributor-avatar"
            >
              <v-avatar size="36" class="elevation-1 border border-opacity-10">
                <v-img :src="c.avatar_url" alt="avatar"></v-img>
              </v-avatar>
            </a>
          </div>
        </div>

      </div>
    </SettingCard>
  </section>
</template>

<script lang="ts" setup>
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'
import SettingCard from '@/components/SettingCard.vue'
import { kEnvironment } from '@/composables/environment'
import { kFlights } from '@/composables/flights'
import { useSWRVConfig } from '@/composables/swrvConfig'
import { vRovingTabindex } from '@/directives/rovingTabindex'
import { vSharedTooltip } from '@/directives/sharedTooltip'
import { injection } from '@/util/inject'
import useSWRV from 'swrv'
import { useId } from 'vue'
import logo from '../assets/logo.webp'

const env = injection(kEnvironment)
const flights = injection(kFlights)

const contributorsHeadingId = useId()

const debugInfo = computed(() => {
  return JSON.stringify({ ...env.value, flights }, null, 2)
})

const { t } = useI18n()
const version = computed(() => env.value?.version ?? '')

const config = useSWRVConfig()
const { data: contributors } = useSWRV<{ login: string; avatar_url: string; html_url: string }[]>(
  'https://api.github.com/repos/voxelum/x-minecraft-launcher/contributors',
  (e) => fetch(e).then((r) => r.json()),
  config,
)
</script>

<style scoped>
.debug-info-code {
  background: rgba(0, 0, 0, 0.25);
  border: var(--card-subsection-border);
  border-radius: var(--card-item-radius);
  color: rgba(var(--v-theme-on-surface), 0.85);
  max-height: 180px;
  overflow-y: auto;
  white-space: pre-wrap;
  word-break: break-all;
}

.contributor-avatar {
  transition: transform 0.2s;
}
.contributor-avatar:hover {
  transform: scale(1.15) translateY(-2px);
  z-index: 1;
}

.sponsor-card {
  background: rgba(var(--v-theme-surface), 0.3);
  border: var(--card-subsection-border);
  border-radius: var(--card-item-radius);
  transition: all 0.2s ease;
  min-height: 120px;
  justify-content: flex-start;
}
.sponsor-card:hover {
  background: rgba(var(--v-theme-surface), 0.5);
  border-color: rgba(var(--v-theme-primary), 0.3);
  transform: translateY(-2px);
}

.sponsor-logo-link {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 36px;
  width: 100%;
}

.sponsor-svg {
  height: 24px;
  max-width: 100%;
  color: rgba(var(--v-theme-on-surface), 0.9);
}

:deep(.v-img__img) {
  object-fit: contain;
}
</style>
