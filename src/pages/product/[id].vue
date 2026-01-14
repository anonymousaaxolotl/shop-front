<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <h1 class="text-center">{{ product.name }}</h1>
        <v-divider></v-divider>
        <v-col cols="12">
          <v-img contain height="400" :src="product.imageUrl"></v-img>
        </v-col>
        <v-col cols="12">
          <v-number-input v-model="quantity" control-variant="split" :min="1"></v-number-input>
          <v-btn
            block
            color="primary"
            prepend-icon="mdi-cart"
            variant="outlined"
            @click="addCart"
          >
            加入購物車
          </v-btn>
        </v-col>
        <v-col class="text-center" cols="12">
          <p>分類: {{ product.category }}</p>
          <p>價格: {{ product.price }}</p>
          <p style="white-space: pre;">{{ product.description }}</p>
        </v-col>
      </v-col>
    </v-row>
  </v-container>
  <v-overlay
    class="align-center justify-center text-center"
    :model-value="!product.sell"
    opacity="0.7"
    persistent
    scrim="black"
  >
    <h1>已下架</h1>
    <v-btn to="/">回首頁</v-btn>
  </v-overlay>
</template>

<script setup>
import { ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useSnackbar } from 'vuetify-use-dialog'
import serviceProduct from '@/services/product'
import serviceUser from '@/services/user'
import { useUserStore } from '@/stores/user'

const route = useRoute()
const router = useRouter()
const createSnackbar = useSnackbar()
const user = useUserStore()

const product = ref({
  name: '',
  price: 0,
  description: '',
  category: '',
  sell: true,
  imageUrl: '',
})

const getProduct = async () => {
  try {
    const { data } = await serviceProduct.getId(route.params.id)
    product.value = data.result
    document.title = `購物網站 | ${product.value.name}`
  } catch (error) {
    console.log(error)
    const text = error?.response?.data?.message || '發生錯誤'
    createSnackbar({
      text,
      snackbarProps: {
        color: 'red',
      },
    })
    router.push('/')
  }
}
getProduct()

const quantity = ref(1)

const addCart = async () => {
  try {
    if (!user.isLoggedIn) {
      router.push('/login')
      return
    }
    const { data } = await serviceUser.cart({ product: product.value.id, quantity: quantity.value, replace: false })
    user.cart = data.result
    createSnackbar({
      text: '加入購物車成功',
      snackbarProps: {
        color: 'green',
      },
    })
  } catch (error) {
    console.log(error)
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
  title: 商品
</route>
