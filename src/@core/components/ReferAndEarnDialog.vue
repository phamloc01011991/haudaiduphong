<script setup lang="ts">
interface Props {
  isDialogVisible: boolean
}

interface Emit {
  (e: 'update:isDialogVisible', val: boolean): void
}

const props = defineProps<Props>()

const emit = defineEmits<Emit>()

const dialogVisibleUpdate = (val: boolean) => {
  emit('update:isDialogVisible', val)
}

const referAndEarnSteps = [
  {
    icon: 'bx-message',
    title: 'Send Invitation 👍🏻',
    subtitle: 'Send your referral link to your friend',
  },
  {
    icon: 'bx-file',
    title: 'Registration 😎',
    subtitle: 'Let them register to our services',
  },
  {
    icon: 'bx-gift',
    title: 'Free Trial  🎉',
    subtitle: 'Your friend will get 30 days free trial',
  },
]
</script>

<template>
  <VDialog
    :model-value="props.isDialogVisible"
    max-width="900"
    @update:model-value="dialogVisibleUpdate"
  >
    <VCard class="refer-and-earn-dialog">
      <!-- 👉 dialog close btn -->
      <DialogCloseBtn
        variant="text"
        size="small"
        @click="emit('update:isDialogVisible', false)"
      />

      <VCardText class="pa-5 pa-sm-13">
        <h5 class="text-h5 text-center mb-3">
          Refer & Earn
        </h5>
        <p class="text-body-2 text-center">
          Ohh it's dosomething
        </p>

        <VRow class="text-center mt-6">
          <VCol
            v-for="step in referAndEarnSteps"
            :key="step.title"
            cols="12"
            sm="4"
          >
            <VAvatar
              variant="tonal"
              size="100"
              color="primary"
            >
              <VIcon
                size="40"
                :icon="step.icon"
              />
            </VAvatar>

            <h6 class="text-base mt-2 mb-3">
              {{ step.title }}
            </h6>
            <span class="text-sm">{{ step.subtitle }}</span>
          </VCol>
        </VRow>
      </VCardText>

      <VDivider />

      <VCardText class="pa-5 pa-sm-13">
        <h6 class="text-h6 mb-4">
          Invite your friends
        </h6>

        <p class="text-sm mb-2">
          Enter your friend's email address and invite them to join Sneat 😍
        </p>
        <VForm
          class="d-flex align-center gap-4"
          @submit.prevent="() => {}"
        >
          <VTextField
            density="compact"
            placeholder="johnDoe@gmail.com"
          />

          <VBtn type="submit">
            Submit
          </VBtn>
        </VForm>

        <h6 class="text-h6 mb-4 mt-8">
          Share the referral link
        </h6>

        <p class="text-sm mb-2">
          You can also copy and send it or share it on your social media. 🚀
        </p>
        <VForm
          class="d-flex align-center flex-wrap gap-3"
          @submit.prevent="() => {}"
        >
          <VTextField
            density="compact"
            placeholder="http://referral.link"
            class="refer-link-input me-1"
          >
            <template #append-inner>
              <VBtn variant="text">
                COPY LINK
              </VBtn>
            </template>
          </VTextField>

          <div class="d-flex gap-3 mx-auto">
            <VBtn
              icon
              class="rounded"
              color="#3B5998"
              size="40"
            >
              <VIcon
                color="white"
                icon="bxl-facebook-circle"
              />
            </VBtn>

            <VBtn
              icon
              class="rounded"
              color="#55ACEE"
              size="40"
            >
              <VIcon
                color="white"
                icon="bxl-twitter"
              />
            </VBtn>

            <VBtn
              icon
              class="rounded"
              color="#007BB6"
              size="40"
            >
              <VIcon
                color="white"
                icon="bxl-linkedin-square"
              />
            </VBtn>
          </div>
        </VForm>
      </VCardText>
    </VCard>
  </VDialog>
</template>

<style lang="scss">
.refer-link-input {
  .v-field--appended {
    padding-inline-end: 0;
  }

  .v-field__append-inner {
    padding-block-start: 0.125rem;
  }
}
</style>
