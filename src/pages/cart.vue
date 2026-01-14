<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <h1 class="text-center">購物車</h1>
      </v-col>
      <v-divider></v-divider>
      <v-col cols="12">
        <v-list lines="two">
          <v-list-item v-for="(item, i) in cart" :key="item._id" :active="!item.product.sell" color="red">
            <template #prepend>
              <v-avatar
                class="cursor-pointer"
                :image="item.product.imageUrl"
                @click="$router.push('/product/' + item.product._id)"
              ></v-avatar>
            </template>
            <v-list-item-title>{{ item.product.name }}</v-list-item-title>
            <v-list-item-subtitle>${{ item.product.price }}</v-list-item-subtitle>
            <template #append>
              <v-number-input
                control-variant="split"
                hide-details
                :min="0"
                :model-value="item.quantity"
                variant="outlined"
                @update:model-value="updateCart($event, item, i)"
              ></v-number-input>
            </template>
          </v-list-item>
        </v-list>
        <p class="text-center">總價: {{ totalPrice }}</p>
        <v-btn color="green" :disabled="checkoutDisable" @click="checkout">結帳</v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { computed, ref } from 'vue'
import { useRouter } from 'vue-router'
import { useSnackbar } from 'vuetify-use-dialog'
import serviceOrder from '@/services/order'
import serviceUser from '@/services/user'
import { useUserStore } from '@/stores/user'

const createSnackbar = useSnackbar()
const user = useUserStore()
const router = useRouter()

const cart = ref([])

const getCart = async () => {
  try {
    const { data } = await serviceUser.getCart()
    cart.value = data.result
  } catch {
    createSnackbar({
      text: '發生錯誤',
      snackbarProps: {
        color: 'red',
      },
    })
  }
}
getCart()

const updateCart = async (value, item, i) => {
  try {
    const { data } = await serviceUser.cart({ product: item.product._id, quantity: value, replace: true })
    user.cart = data.result
    if (value <= 0) {
      cart.value.splice(i, 1)
    } else {
      cart.value[i].quantity = value
    }
  } catch {
    const text = error?.response?.data?.message || '發生錯誤'
    createSnackbar({
      text,
      snackbarProps: {
        color: 'red',
      },
    })
  }
}

const totalPrice = computed(() => cart.value.reduce((total, item) => total + (item.product.price * item.quantity), 0))
const checkoutDisable = computed(() => cart.value.length === 0 || cart.value.some(item => !item.product.sell))

const checkout = async () => {
  try {
    const { data } = await serviceOrder.create()
    user.cart = 0
    router.push('/orders')
    createSnackbar({
      text: `結帳成功，訂單 ID: ${data.result._id}`,
      snackbarProps: {
        color: 'green',
      },
    })
  } catch {
    const text = error?.response?.data?.message || '發生錯誤'
    createSnackbar({
      text,
      snackbarProps: {
        color: 'red',
      },
    })
  }
}
</script>

<route lang="yaml">
meta:
  title: 購物車
  login: login-only
</route>
