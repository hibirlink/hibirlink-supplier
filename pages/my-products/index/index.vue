<template>
  <main class="p-2">
    <h-bread-crumb
      bread-crumb-class="bg-light50"
      home-icon-class="fill-dark300"
    />
    <h-empty
      v-if="!loading && !storeData.length"
      content="You Have No Store Associated"
    >
      <template #image>
        <img
          src="https://th.bing.com/th/id/R.432cb178751aebf76b9166cdc4723c78?rik=vahYnC5T2xe7WA&riu=http%3a%2f%2f2.bp.blogspot.com%2f-UomRXLjNZIY%2fTrQDW38PwAI%2fAAAAAAAABQk%2f-_OjDXIr3Yw%2fs1600%2fempty_cart3.jpg&ehk=Vi62%2b1pKvxonVpsGH4dEM6KklKxnauYKJ4iuDHt%2b%2fGQ%3d&risl=&pid=ImgRaw&r=0"
          alt=""
          srcset=""
          class="w-[200px] h-[200px] object-cover object-center"
        />
      </template>
    </h-empty>
    <div
      v-else
      class="grid grid-cols-1 sm:grid-cols-2 xl:grid-cols-3 2xl:grid-cols-4 gap-x-6 gap-y-5 mt-5 mb-16"
    >
      <card-store
        v-for="store in storeData"
        :key="store.id"
        :id="store.id"
        :storeName="store.name"
        :status="store.status"
        :products="store.total_product.total.count"
        :employee="store?.total?.employee?.value"
        :date="store.created_at.slice(0, 10)"
        :business="store.category"
        :bank="store.bank"
        @menu-selected="handleMenu"
        @click="handleClickStore(store.id)"
      />
    </div>

    <div class="w-full h-8"></div>
    <h-button
      v-if="role === 'supplier'"
      btn-name="add_store"
      type="button"
      class="w-fit fixed bottom-20 right-7"
      leading-icon="uil:instagram"
      :gradient="false"
      rounded-class="rounded-md"
      leading-icon-class="bg-primary"
      btn-class="overflow-hidden bg-darkblue text-light400"
      btn-name-class="px-2 font-bold"
      @click="addStore"
    />
  </main>
</template>
<script setup>
import fetchStoreQl from "@/apollo/query/product/realTime/fetchStore.gql";
import fetchEmployeeStoreQl from "@/apollo/query/product/realTime/fetchEmployeeStore.gql";
import { storeToRefs } from "pinia";
import { useAuth } from "~~/store/auth";

const { uid, role, manager_id } = storeToRefs(useAuth());
const employee = ref(role.value == "gust" || role.value == "store_manager");
const {
  result: storeResult,
  onError: storeError,
  loading,
} = useCustomQuery(fetchStoreQl, { supplier_id: uid.value });

console.log("ROLE", role.value, manager_id);

storeError((error) => {
  console.log(error);
});
const storeData = computed(() => {
  console.log(storeResult.value?.supplier.stores);

  return storeResult.value?.supplier.stores ?? [];
});

const stores = ref([]);
const router = useRouter();
const addStore = () => {
  router.push({ name: "my-products-add_store" });
};

const handleMenu = (value) => {
  if (value.action == "edit") {
    router.push({
      name: "my-products-edit_store-storeId",
      params: { storeId: value.id },
    });
  } else if (value.action == "delete") {
  }
};
const handleClickStore = (value) => {
  useRouter().push({
    name: "my-products-index-realTime-storeId",
    params: { storeId: value },
  });
};
</script>
