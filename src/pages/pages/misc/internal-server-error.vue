<script setup lang="ts">
import { useGenerateImageVariant } from '@/@core/composable/useGenerateImageVariant'
import pageMiscErrorDark from '@images/pages/page-misc-error-dark.png'
import pageMiscErrorLight from '@images/pages/page-misc-error-light.png'

const pageMiscError = useGenerateImageVariant(pageMiscErrorLight, pageMiscErrorDark)
</script>

<template>
  <div class="misc-wrapper">
    <ErrorHeader
      error-title="Internal server error 🔐"
      error-description="Oops something went wrong."
    />

    <!-- 👉 Image -->
    <div class="misc-avatar w-100 text-center">
      <VImg
        :src="pageMiscError"
        alt="Coming Soon"
        :max-width="500"
        class="mx-auto"
      />
      <VBtn
        to="/"
        class="mt-10"
      >
        Back to Home
      </VBtn>
    </div>
  </div>
</template>

<style lang="scss">
@use "@core/scss/template/pages/misc.scss";
</style>

<route lang="yaml">
meta:
  layout: blank
</route>
