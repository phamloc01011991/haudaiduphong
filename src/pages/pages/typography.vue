<script setup lang="ts">
import { notify } from "@kyvg/vue3-notification";
import { languagePack } from "@/languages";
import request from "@/utils2/request";
import { useUserListStore } from "@/views/apps/user/useUserListStore";
import "ant-design-vue/dist/antd.css";
import { computed, onMounted, ref, watch, watchEffect } from "vue";

// 👉 Store
const userListStore = useUserListStore();
const searchQuery = ref("");
const rowPerPage = ref(5); // page size
const currentPage = ref(1);
const totalUsers = ref(14);
const totalPage = ref(1);

const users = ref<any>([]);
const nameUser = ref("");
const idUser = ref("");
const loadingTable = ref(false);
const loadingVerifyUser = ref<any>(0);
const cnyAmount = ref();

watch(currentPage, (newVal, oldVal) => {
  reLoadOrder(newVal);
});

// 👉 Test

async function usdToCny() {
  const reponse = await fetch(
    "https://api.exchangerate-api.com/v4/latest/USDT"
  );
  const data = await reponse.json();
  const usdToCnyRate = data.rates.CNY;

  // Thực hiện chuyển đổi từ USDT sang VND
  // const usdtAmount = number; // Số tiền USDT cần chuyển đổi
  // cnyAmount.value = usdtAmount * usdToCnyRate;

  return (cnyAmount.value = usdToCnyRate);
}
usdToCny();

// 👉 watching current page
watchEffect(() => {
  if (currentPage.value > totalPage.value) currentPage.value = totalPage.value;
});

// 👉 search filters
const realSearch = ref();
const selectedOption = ref({ state: "Name user", abbr: "name" });

watch(selectedOption, (newVal, oldVal) => {
  realSearch.value = newVal;
});

const chooseSearch = [
  { state: "Name", abbr: "name" },
  { state: "ID", abbr: "id" },
];

window.addEventListener("keyup", (event) => {
  if (event.keyCode === 13) SearchUser();
});
async function SearchUser() {
  try {
    if (realSearch.value == "name") {
      idUser.value = "";
      nameUser.value = searchQuery.value;
      reLoadOrder();
    }
    if (realSearch.value == "id") {
      nameUser.value = "";
      idUser.value = searchQuery.value;
      reLoadOrder();
    } else {
      idUser.value = "";
      nameUser.value = searchQuery.value;
      reLoadOrder();
    }
  } catch (error) {}
}

// 👉 watching current page
watchEffect(() => {
  if (currentPage.value > totalPage.value) currentPage.value = totalPage.value;
});

// 👉 Convert To Currency

// 👉 Computing pagination data
const paginationData = computed(() => {
  const firstIndex = users.value.length
    ? (currentPage.value - 1) * rowPerPage.value + 1
    : 0;

  const lastIndex =
    users.value.length + (currentPage.value - 1) * rowPerPage.value;

  return `${firstIndex}-${lastIndex} of ${totalUsers.value}`;
});

// SECTION Checkbox toggle
const selectedRows = ref<string[]>([]);
const selectAllUser = ref(false);

// 👉 watch if checkbox array is empty all select should be uncheck

watch(
  selectedRows,
  () => {
    if (!selectedRows.value.length) selectAllUser.value = false;
  },
  { deep: true }
);

// !SECTION checkbox toggle

// Khai báo biến
const lstOrderLoad = ref<any>([]);
const totalKyc = ref();

// const limitPage = ref(10);
// lệnh kiểm soat
const reLoadOrder = async (page = 1) => {
  try {
    loadingTable.value = true;

    const res = await request.get("listingKyc", {
      params: {
        page,
        limit: rowPerPage.value,
        nameUser: nameUser.value,
        id: idUser.value,
      },
    });

    if (res.data.success === true) {
      loadingTable.value = false;

      const data = res.data;

      lstOrderLoad.value = data.data;
      totalKyc.value = data.totalCount;
      totalPage.value = Math.ceil(totalKyc.value / rowPerPage.value);

      // console.error("Failed to fetch dynamically imported module")
    }
  } catch (error) {
    loadingTable.value = false;
    console.log(error);
  }
};

const verifyAccount = async (idUser: number, idKyc: number, status: number) => {
  loadingVerifyUser.value = idKyc;

  const res = await request.post("admin/verifyAccount", {
    idUser,
    idKyc,
    status,
  });

  loadingVerifyUser.value = 0;
  if (res.data.success == true) {
    notify({
      title: "Thành công",
      text: "Cập nhật thành công!",
      type: "success",
    });
    reLoadOrder();
  } else {
    notify({
      title: "Thất bại",
      text: "Cập nhật thất bại!",
      type: "error",
    });
  }
};

onMounted(() => {
  reLoadOrder();
});

// -----
</script>

<template>
  <section>
    <VRow>
      <VCol cols="12">
        <VCard :title="languagePack.VERY_USER_TITLE">
          <VCardText>
            <VRow>
              <!-- 👉 Search for.... -->
              <VCol cols="12" sm="4">
                <div style="height: 100%">
                  <VSelect
                    v-model="selectedOption"
                    :label="languagePack.VERY_USER_ORDER_BY"
                    :items="chooseSearch"
                    item-title="state"
                    item-value="abbr"
                  />
                </div>
              </VCol>
              <!-- 👉 Search form -->
              <VCol cols="12" sm="4">
                <div
                  class="app-user-search-filter d-flex align-center"
                  style="height: 100%"
                >
                  <!-- 👉 Search  -->
                  <VTextField
                    v-model="searchQuery"
                    :placeholder="languagePack.VERY_USER_SEARCHKEY_PLACEHOLDER"
                    density="compact"
                    class="me-3"
                  />

                  <!-- <VBtn> <VIcon start icon="bx-search" />Search </VBtn> -->
                  <VBtn @click="SearchUser">
                    {{ languagePack.VERY_USER_SEARCH_BUTTON }}
                    <VIcon end icon="bx-search" />
                  </VBtn>

                  <!--
                    <VBtn @click="toggleAutoFetchData">{{ autoFetchData ? 'Tắt' : 'Bật' }}
                    <VIcon end icon="bx-search" />
                    </VBtn>
                  -->
                </div>
              </VCol>
            </VRow>
          </VCardText>

          <VDivider />

          <VCardText class="d-flex flex-wrap gap-4">
            <!-- 👉 Export button -->
            <VBtn
              variant="tonal"
              color="secondary"
              prepend-icon="bx-arrow-from-bottom"
            >
              {{ languagePack.VERY_USER_EXPORT }}
            </VBtn>

            <VSpacer />

            <!-- <div class="app-user-search-filter d-flex align-center"> -->
            <!--
              👉 Search
              <VTextField
              v-model="searchQuery"
              placeholder="Search User"
              density="compact"
              class="me-3"
              />
            -->

            <!-- 👉 Add user button -->
            <!-- </div> -->
          </VCardText>

          <VDivider />
          <VProgressLinear
            v-if="loadingTable"
            height="10"
            indeterminate
            color="primary"
          />
          <VTable class="text-no-wrap">
            <!-- 👉 table head -->
            <thead>
              <tr>
                <th scope="col">ID</th>
                <th scope="col">
                  {{ languagePack.VERY_USER_ID_USER }}
                </th>
                <!-- <th scope="col">
                  {{ languagePack.USER_MANAGE_username }}
                </th> -->
                <th scope="col">
                  {{ languagePack.VERY_USER_REAL_NAME }}
                </th>
                <th scope="col">
                  {{ languagePack.ID_PASSPORT }}
                </th>
                <th scope="col">
                  {{ languagePack.VERY_USER_FRONT }}
                </th>
                <th scope="col">
                  {{ languagePack.VERY_USER_BACK }}
                </th>
                <th scope="col">
                  {{ languagePack.VERY_USER_ACTIONS }}
                </th>
              </tr>
            </thead>

            <!-- 👉 table body -->
            <tbody>
              <tr v-for="user in lstOrderLoad" :key="user.id">
                <td>
                  {{ user.id }}
                </td>
                <td>
                  {{ user.userId }}
                </td>
                <!-- 👉 User -->
                <!-- <td>
                  <div class="d-flex align-center">
                    <div class="d-flex flex-column">
                      <span class="text-base">
                        {{ user.fullName }}
                      </span>
                    </div>
                  </div>
                </td> -->
                <td>
                  <span class="text-capitalize text-base">
                    {{ user.real_name || "Lê Quốc Thuận" }}</span
                  >
                </td>
                <td>
                  <span class="text-capitalize text-base"
                    >{{ user.identity_card || "182464804" }}
                  </span>
                </td>
                <td>
                  <span class="text-capitalize text-base">
                    <AImage :width="100" :src="user.frontImage" />
                  </span>
                </td>

                <!-- 👉 Role -->
                <td>
                  <span class="text-capitalize text-base">
                    <!-- <img width="100" :src="user.backImage" alt=""> -->

                    <AImage :width="100" :src="user.backImage"
                  /></span>
                </td>

                <!-- 👉 Actions -->
                <td
                  v-if="user.status == 'pending'"
                  class="text-center"
                  style="width: 80px"
                >
                  <!-- <MoreBtn :menu-list="computedMoreList(user.userId)" item-props /> -->
                  <VBtn
                    color="success"
                    size="small"
                    :loading="user.id === loadingVerifyUser ? true : false"
                    @click="verifyAccount(user.userId, user.id, 3)"
                  >
                    {{ languagePack.VERY_USER_ACTION_APPROVED }} </VBtn
                  >&ensp;
                  <VBtn
                    color="error"
                    size="small"
                    :loading="user.id === loadingVerifyUser ? true : false"
                    @click="verifyAccount(user.userId, user.id, 1)"
                  >
                    {{ languagePack.VERY_USER_ACTION_CANCEL }}
                  </VBtn>
                </td>
                <td v-else class="text-center" style="width: 80px">
                  <VBtn
                    v-if="user.status == 'approved'"
                    color="success"
                    variant="text"
                    size="small"
                  >
                    {{ languagePack.VERY_USER_APPROVED }}
                  </VBtn>
                  <VBtn v-else color="error" variant="text" size="small">
                    {{ languagePack.VERY_USER_REJECTED }}
                  </VBtn>
                </td>
              </tr>
            </tbody>
          </VTable>

          <VDivider />

          <!-- SECTION Pagination -->
          <VCardText class="d-flex flex-wrap justify-end gap-4 pa-2">
            <!-- 👉 Rows per page -->
            <div class="d-flex align-center" style="width: 171px">
              <span class="text-no-wrap text-sm me-3">Rows per page:</span>
              <VSelect
                v-model="rowPerPage"
                density="compact"
                class="per-page-select"
                variant="plain"
                :items="[10, 20, 30, 50]"
              />
            </div>

            <!-- 👉 Pagination and pagination meta -->
            <div class="d-flex align-center">
              <h6 class="text-sm font-weight-regular">
                {{ paginationData }}
              </h6>
            </div>
            <VPagination
              v-model="currentPage"
              :total-visible="1"
              rounded="circle"
              :length="totalPage"
              @next="selectedRows = []"
              @prev="selectedRows = []"
            />
          </VCardText>
          <!-- !SECTION -->
        </VCard>
      </VCol>
    </VRow>
    <!-- 👉 Add New User -->
  </section>
</template>

<style lang="scss" scoped>
//
#app
  > div.v-locale-provider.v-locale--is-ltr
  > div
  > div
  > div
  > div.layout-content-wrapper
  > main
  > div
  > section
  > div
  > div.v-col.v-col-12
  > div
  > div:nth-child(3)
  > div
  > div:nth-child(1)
  > div
  > div
  > div {
  height: 40px;
}
.app-user-search-filter {
  inline-size: 385px;
}

.text-capitalize {
  text-transform: capitalize;
}

.user-list-name:not(:hover) {
  color: rgba(var(--v-theme-on-background), var(--v-high-emphasis-opacity));
}

.per-page-select {
  margin-block: auto;

  .v-field__input {
    align-items: center;
    padding: 2px;
    font-size: 14px;
  }

  .v-field__append-inner {
    align-items: center;
    padding: 0;
    margin-inline-start: -2.5rem;

    .v-icon {
      margin-inline-start: 0 !important;
    }
  }
}
#app
  > div.v-locale-provider.v-locale--is-ltr
  > div
  > div
  > div
  > div.layout-content-wrapper
  > main
  > div
  > section
  > div
  > div.v-col.v-col-12
  > div
  > div:nth-child(3)
  > div
  > div:nth-child(1)
  > div
  > div
  > div
  > div
  > div.v-field__field {
  height: 40px;
  position: relative;
  top: -5px;
}
#app
  > div.v-locale-provider.v-locale--is-ltr
  > div
  > div
  > div
  > div.layout-content-wrapper
  > main
  > div
  > section
  > div
  > div.v-col.v-col-12
  > div
  > div:nth-child(3)
  > div
  > div:nth-child(1)
  > div
  > div
  > div
  > div
  > div.v-field__append-inner {
  position: relative;
  bottom: 6px;
}
#app
  > div.v-locale-provider.v-locale--is-ltr
  > div
  > div
  > div
  > div.layout-content-wrapper
  > main
  > div
  > section
  > div
  > div.v-col.v-col-12
  > div
  > div:nth-child(3)
  > div
  > div:nth-child(1)
  > div
  > div
  > div
  > div
  > div.v-field__clearable {
  position: relative;
  bottom: 6px;
}
</style>
