<template>
  <main
    class="flex justify-center items-start gap-6 max-w-[90rem] min-h-full mx-auto p-5"
  >
    <div class="flex-1 sm:min-w-[28rem] lg:max-w-md max-w-full space-y-2">
      <h-text-field
        name="search"
        placeholder="Search"
        leading-icon="ph:magnifying-glass"
      />
      <div class="space-y-4 h-[calc(100vh-9.3rem)] overflow-auto">
        <NuxtLink
          :to="{ path: `/chats`, query: { id: id } }"
          class="flex items-center gap-x-4 cursor-pointer h-fit p-2 rounded w-full"
          v-for="id in Object.keys(messages)"
          :key="id"
          :class="[
            activeChat == id ? 'bg-primaryvar4' : 'hover:bg-primaryvar7',
          ]"
        >
          <div class="relative flex-shrink-0">
            <span
              class="absolute bottom-0 right-0 w-4 h-4 bg-lemon border rounded-full text-neutral400 border-light100"
            ></span>
            <img
              src="https://source.unsplash.com/50x50/?portrait"
              alt=""
              class="w-14 h-14 border rounded-full bg-neutral300 border-neutral500"
            />
          </div>

          <div class="h-full flex flex-col">
            <div class="font-bold text-dark100 line-clamp-1">
              {{ messages[id].name }}
            </div>
            <div class="text-neutral200 font-medium line-clamp-1">
              {{ messages[id].message }}
            </div>
          </div>
          <div class="flex flex-col justify-between items-end h-full ml-auto">
            <p class="text-neutral200 font-medium text-md truncate">
              {{ messages[id].time }}
            </p>
            <div
              v-if="messages[id].notification"
              class="bg-primary w-fit h-fit px-2 rounded-full font-medium text-light400 text-md"
            >
              1
            </div>
          </div>
        </NuxtLink>
      </div>
    </div>
    <div class="flex-auto min-h-full" v-if="activeChat"><NuxtPage /></div>
    <div class="flex-auto min-h-full" v-else></div>
  </main>
</template>

<script setup lang="ts">
import fetchChatUsersGql from "@/apollo/query/chat/fetchChatUsers.gql";
import { reactive, watchEffect } from "vue";
import { useAuth } from "~~/store/auth";
const { uid } = useAuth();
const { result, onError } = useCustomQuery(fetchChatUsersGql, {
  user_id: uid,
});
onError((error) => {
  console.log(error);
});

const allMessage = computed(() => result.value?.message);

watchEffect(() => {
  // console.log("-------------sfs-df-sdf-sd-f-d");
  // console.log(allMessage.value);

  allMessage.value?.forEach((msg: any) => {
    if (msg.sender_id === uid) {
      messages[msg.receiver_id] = {
        message: msg.message,
        name: msg.user.first_name + " " + msg.user.last_name,
        time: msg.created_at,
        notification: false,
      };
    } else if (msg.receiver_id === uid) {
      messages[msg.sender_id] = {
        message: msg.message,
        name: msg.userBySenderId.first_name + " " + msg.user.last_name,
        time: msg.created_at,
        notification: !msg.seen,
      };
    }
  });
});

const messages = <any>reactive({});

allMessage.value?.forEach((msg: any) => {
  if (msg.sender_id === uid) {
    messages[msg.receiver_id] = {
      message: msg.message,
      name: msg.user.first_name + " " + msg.user.last_name,
      time: msg.created_at,
      seen: msg.seen,
    };
  } else if (msg.receiver_id === uid) {
    messages[msg.sender_id] = {
      message: msg.message,
      name: msg.userBySenderId.first_name + " " + msg.user.last_name,
      time: msg.created_at,
      seen: msg.seen,
    };
  }
});

watchEffect(() => {
  console.log("-------------------");
  console.log(messages);
  console.log("-------------------");
});

const chatPersons = computed(() => {
  const users = [];

  if (result.value?.message) {
    const msg = result.value?.message;
    users.push(msg[0]);
    var outLoop = true;
    for (let x = 1; x < msg.length; x++) {
      for (let y = 0; y < users.length; y++) {
        if (
          (msg[x].sender_id == users[y].sender_id ||
            msg[x].sender_id == users[y].receiver_id) &&
          (msg[x].receiver_id == users[y].sender_id ||
            msg[x].receiver_id == users[y].receiver_id)
        ) {
          console.log("x = ", x, "------>", msg[x], users[y]);

          outLoop = false;
          const date1 = new Date(msg[x].created_at);
          const date2 = new Date(users[y].created_at);
          if (date1 > date2) {
            users[y] = msg[x];
          }
        }
      }
      if (outLoop) {
        users.push(msg[x]);
        outLoop = true;
      }
    }

    const userData = users.map((usr) => {
      return {
        message: usr.message,
        id: usr.sender_id == uid ? usr.receiver_id : usr.sender_id,
        name:
          usr.userBySenderId.id == uid
            ? usr.user.first_name + " " + usr.user.last_name
            : usr.userBySenderId.first_name +
              " " +
              usr.userBySenderId.last_name,
        time: usr.created_at.substring(0, 16).split("T"),
        notification: usr.receiver_id == uid && !usr.seen,
      };
    });

    return userData;
  }
  return null;
});

const route: any = useRoute();
const activeChat = computed(() => route?.query?.id ?? null);
useHead({
  title: "Chat",
});

definePageMeta({
  layout: "chat",
});
</script>
