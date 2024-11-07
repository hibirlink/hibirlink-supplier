<template>
  <div class="my-5 mb-16">
    <h-notification
      @close="messageShow = false"
      :success="messageStatus"
      v-model="messageShow"
      >{{ messageContent }}
    </h-notification>
    <h-confirm-action
      v-if="showConfirmationModal"
      @confirmation="confirmationAction"
    />
    <h-empty
      v-if="!loading && !products.length"
      content="You Do not have any virtual product"
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
    <div v-else v-for="product in products" :key="product.id" class="my-6">
      <card-virtual-product
        :id="product.id"
        :title="product.title"
        :score="product.score"
        :images="product.images"
        :price="product.price"
        :demo_link="product.demo_link"
        :description="product.description"
        @menu-selected="handleSelectedMenu"
      />
    </div>
    <h-button
      v-if="role === 'store_manager' || role === 'supplier'"
      @click="addProduct"
      type="button"
      btn-name="add_product"
      class="w-fit fixed bottom-5 right-7"
      leading-icon="uil:instagram"
      :gradient="false"
      rounded-class="rounded-md"
      leading-icon-class="bg-primary"
      btn-class="overflow-hidden bg-darkblue text-light400"
      btn-name-class="px-2 font-bold"
    />
  </div>
</template>
<script setup lang="ts">
import fetchProducts from "@/apollo/query/product/virtual/fetchProducts.gql";
import deleteProducts from "@/apollo/mutation/product/virtual/deleteProduct.gql";
import { storeToRefs } from "pinia";
import { watchEffect } from "vue";
import { useAuth } from "~~/store/auth";

const { uid, role } = storeToRefs(useAuth());
const router = useRouter();
const route = useRoute();
const { result, error, loading, refetch } = useCustomQuery(fetchProducts, {
  supplier_id: uid.value,
});

watchEffect(() => console.log("--->", result.value, error.value));

const products = computed(() => {
  const data = result.value?.supplier?.products;
  if (data) {
    return data.map((element: any) => {
      var imageData = [];
      imageData.push({ type: "images", url: element.cover_image });
      var description = "";

      element.description.ops.map((txt: any) => {
        description += txt.insert + " ";
      });
      if (element.images) {
        element.images.map((img: any) => {
          imageData.push({ type: "image", url: img.url });
        });
      }
      return {
        id: element.id,
        title: element.title,
        score: element.total_rate.rate.avg.rate,
        images: imageData,
        price: element.unit_price,
        demo_link: element.demo_link,
        description: description,
      };
    });
  }
  return [];
});
const addProduct = () => {
  router.push({
    name: "my-products-add_products-virtual",
  });
};
interface Option {
  id: number;
  action: string;
}
const handleSelectedMenu = (value: Option) => {
  if (value.action == "edit") {
    router.push({
      name: "my-products-edit_product-virtual-productId",
      params: { productId: value.id },
    });
  } else if (value.action == "delete") {
    productToDeleteId.value = value.id;
    showConfirmationModal.value = true;
  }
};
const messageStatus = ref(false);
const messageShow = ref(false);
const messageContent = ref("");
const showNotification = (status: boolean, message: string) => {
  messageShow.value = true;
  messageStatus.value = status;
  messageContent.value = message;
};
const {
  mutate: deleteProduct,
  onDone: onDeleteComplete,
  onError: onDeleteError,
} = useCustomMutation(deleteProducts);
onDeleteComplete((res) => {
  refetch();
  productToDeleteId.value = null;
  showNotification(true, "Product Delete Success");
});

onDeleteError((error) => {
  console.log(error);
  productToDeleteId.value = null;
  showNotification(
    false,
    error.message.includes("foreign")
      ? "U cant delete this product as it is occupied"
      : "Product Delete Error"
  );
});
const productToDeleteId = ref();
const showConfirmationModal = ref(false);
const confirmationAction = (response: any) => {
  showConfirmationModal.value = false;
  if (response.accept) {
    deleteProduct({ product_id: productToDeleteId.value });
  } else {
    productToDeleteId.value = null;
  }
};
</script>
