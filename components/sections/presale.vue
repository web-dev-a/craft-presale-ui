<template>
<section class="ftco-section">
<div class="container">
<div class="row justify-content-center">
<div class="col-md-6 text-center mb-5">
<h2 class="heading-section">Boxicon</h2>
</div>
</div>
<div class="row justify-content-center">
<div class="col-md-6 col-lg-4">
<div class="login-wrap p-0">
<h3 class="mb-4 text-center">{{ remainingMintable }} remaining</h3>

<div class="form-group">
<input
            type="number"
            class="form-control"
            style="font-size: 1.75rem; text-align: center;"
            v-model="v$.mintNumber.$model"
            :errors="v$.mintNumber.$errors"
            :min="0"
            :max="remainingMintable"
            placeholder="Amount"
          />
</div>
<div class="form-group">
        <ControlsButtonAction
          type="submit"
          class="form-control btn btn-primary submit px-3"
          :is-locked="!isLive || (isWhitelistEnabled && !isWhitelisted)"
          @click="presaleMintOnClick"
        >
          Mint
        </ControlsButtonAction>
</div>
<div class="form-group d-md-flex">
<div class="w-50">
<label>
</label>
</div>
<div class="w-50 text-md-right">Total

              <UtilsIcon
                name="Logo/Icon"
                class="w-12 h-12 text-[#00AC97]"
              /> {{ formStates.mintNumber * price }} </div>
</div>

<p class="w-100 text-center">&mdash; Reach us &mdash;</p>
<div class="row justify-content-center">
<a href="https://discord.com/invite/nQz7TxWR4W"><img src="/discord-icon.png" alt="Discord" width="50" height="50" style="margin: 15px;"></a>
<a href="https://twitter.com/olo_pics"><img src="/twitter-icon.png" alt="Twitter" width="50" height="50" style="margin: 15px;"></a>
</div>
</div>
</div>
</div>
</div>
</section>
</template>

<script setup lang="ts">
import { storeToRefs } from 'pinia'
import useVuelidate from '@vuelidate/core'
import { required, decimal } from '@vuelidate/validators'
import { useImagesStore } from '@/stores/images'
import { useUserStore } from '@/stores/user'

type FormStates = {
  mintNumber: number
}

type FormValidators =
  | typeof required
  | typeof decimal

type FormRules = {
  [key in keyof FormStates]: Record<string, FormValidators>
}

const { collection } = useRuntimeConfig()

const { emit, events } = useEventsBus()
const { notify } = useNotificationToast()
const { SCORECallReadOnly } = useScoreService()
const { images } = storeToRefs(useImagesStore())
const { isLoggedIn, address } = storeToRefs(useUserStore())

const totalMintable = ref<number>(0)
const remainingMintable = ref<number>(totalMintable.value)
const progress = ref<number>(0)
const price = ref<number>(0)
const isLoaded = ref<boolean>(false)
const isLive = ref<boolean>(false)
const isWhitelisted = ref<boolean>(false)
const isWhitelistEnabled = ref<boolean>(false)
const mintLimit = ref<number>(0)
const userMintCount = ref<number>(0)

const formStates = reactive<FormStates>({
  mintNumber: null,
})

const v$ = useVuelidate<FormStates, FormRules>({
  mintNumber: { required, decimal },
}, formStates)

const currentImage = computed<string>(() => images.value.unrevealed)

const checkPresale = async (): Promise<void> => {
  try {
    isLive.value = await SCORECallReadOnly('presaleOpened') !== '0x0'

    if (isLive.value) {
      totalMintable.value = 5000
      remainingMintable.value = totalMintable.value - parseInt(await SCORECallReadOnly('mintId'), 16)
      progress.value = (remainingMintable.value / totalMintable.value) * 100
      price.value = parseInt(await SCORECallReadOnly('presalePrice'), 16) / (10 ** 18)
      mintLimit.value = parseInt(await SCORECallReadOnly('mintLimit'), 16)

      if (isLoggedIn.value) {
        isWhitelistEnabled.value = await SCORECallReadOnly('requireWhitelist') !== '0x0'
        isWhitelisted.value = await SCORECallReadOnly('isWhitelisted', { _address: address.value }) !== '0x0'
        userMintCount.value = parseInt(await SCORECallReadOnly('mintCount', { _address: address.value }), 16)
      }
    }
  } catch (error) {
    throw new Error(error)
  } finally {
    isLoaded.value = true
  }
}

const presaleMintOnClick = (): void => {
  if (!isLoggedIn.value) {
    notify.error({
      title: 'Error',
      message: 'You need to log in first.',
      timeout: 5000,
    })
  } else if (mintLimit.value > 0 && (userMintCount.value + formStates.mintNumber) >= mintLimit.value) {
    notify.error({
      title: 'Error',
      message: 'You exceed the mint limit.',
      timeout: 5000,
    })
  } else if (formStates.mintNumber > 200) {
    notify.error({
      title: 'Error',
      message: `You can only mint 200 ${collection} per transaction.`,
      timeout: 5000,
    })
  } else if (!formStates.mintNumber) {
    notify.error({
      title: 'Error',
      message: `You must mint at least 1 ${collection} to purchase.`,
      timeout: 5000,
    })
  } else if (formStates.mintNumber > remainingMintable.value) {
    notify.error({
      title: 'Error',
      message: `Not enough ${collection} remaining.`,
      timeout: 5000,
    })
  } else if (formStates.mintNumber <= remainingMintable.value) {
    emit(events.POPUP_ACTION, {
      name: 'PresaleMint',
      params: {
        amount: formStates.mintNumber,
        price: price.value,
        image: currentImage,
      },
      handleGuard: true,
    })
  }
}

onMounted(async () => {
  await checkPresale()
})
</script>
