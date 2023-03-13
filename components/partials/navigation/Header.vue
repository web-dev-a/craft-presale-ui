<template>
  <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
  <link href="https://fonts.googleapis.com/css?family=Lato:300,400,700&display=swap" rel="stylesheet">
  <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css">
  <link rel="stylesheet" href="/style.css">
  <header class="sticky top-0 py-10">
<div class="container">
<div class="row justify-content-center" style="margin: auto;">
<div class="gap-12 grid-flow-col items-center justify-self-end">

        <client-only>
          <template v-if="isLoggedIn && truncatedAddress">
            <ControlsButtonAction type="submit" class="form-control btn btn-primary submit px-3" @click="emit(events.POPUP_GUARD)">
              {{ truncatedAddress }}
            </ControlsButtonAction>
          </template>
          <ControlsButtonAction type="submit" class="form-control btn btn-primary submit px-3"
            v-else
            @click="emit(events.POPUP_GUARD)"
          >
            Connect wallet
          </ControlsButtonAction>
        </client-only>
      </div>
      </div>
      </div>
  </header>
</template>
<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { useImagesStore } from '@/stores/images'
import { useUserStore } from '@/stores/user'
import type { IconsNames } from '@/composables/useIconsComponents'
type ExternalLink = {
  name: string
  url: string
}
type SocialLink = {
  icon: IconsNames
  url: string
  color: string
}
const { images } = storeToRefs(useImagesStore())
const { isLoggedIn, truncatedAddress } = storeToRefs(useUserStore())
const { emit, events } = useEventsBus()
const { collection, scoreAddress } = useRuntimeConfig()
const isMenuOpen = ref<boolean>(false)
const title = ref<string>(collection)
const externalLinks = ref<ExternalLink[]>([
  { name: 'Guide', url: 'https://medium.com/@craftnetwork/tutorial-how-to-mint-an-nft-on-a-launchpad-presale-671914a5b3dd' },
  { name: 'Marketplace', url: `https://craft.network/collection/${scoreAddress}` },
])
const socialLinks = ref<SocialLink[]>([
  { icon: 'Logo/Discord', url: 'https://discord.gg/nQz7TxWR4W', color: '#667CD3' },
  { icon: 'Logo/Twitter', url: 'https://twitter.com/olo_pics', color: '#6FC4FE' },
])
</script>
