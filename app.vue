<template>
  <h-loading v-if="loading" :show-loading="loading" />
  <main v-else class="min-w-screen min-h-screen font-nunito">
    <NuxtLayout>
      <NuxtPage />
    </NuxtLayout>
  </main>
</template>
<script setup lang="ts">
import fetchSupplierStatus from "@/apollo/query/fetchSupplierStatus.gql";
import useCustomQuery from "./composables/useCustomQuery";
import { useAuth } from "./store/auth";

const router = useRouter();
const loading = ref(true);

const { uid, setReviewMessage, setReviewStatus, role } = useAuth();

if (role === "supplier" || role === "guest" || role === "store_manager") {
} else {
  // window.location.href = "http://localhost:3000";
}

const { onResult, onError } = useCustomQuery(fetchSupplierStatus, { uid });

onResult(async (data: any) => {
  if (data.data.supplier_by_pk) {
    if (
      data.data.supplier_by_pk.userById.status === "pending" ||
      data.data.supplier_by_pk.userById.status === "suspended"
    ) {
      setReviewMessage(data.data.supplier_by_pk.userById.reason);
      setReviewStatus(data.data.supplier_by_pk.userById.status);
      loading.value = false;
      await router.replace({ name: "review" });
    } else if (data.data.supplier_by_pk.userById.status === "verified") {
      loading.value = false;
    } else {
      loading.value = false;
      window.location.href = "http://localhost:3000";
    }
  } else {
    loading.value = false;
    window.location.href = "http://localhost:3000";
  }
});
onError((error) => {
  console.log(error);
  loading.value = false;
});
</script>
