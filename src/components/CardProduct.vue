<template>
  <v-card>
    <v-img class="align-end text-white" cover height="200" :src="imageUrl">
    </v-img>
    <v-card-title>
      <router-link :to="'/product/' + id">{{ name }}</router-link>
    </v-card-title>
    <v-card-subtitle>
      {{ category }} / {{ formatedPrice }}
    </v-card-subtitle>
    <v-card-text>
      {{ description }}
    </v-card-text>
    <v-card-actions>
      <v-btn
        block
        color="primary"
        prepend-icon="mdi-cart"
        variant="outlined"
        @click="addCart"
      >加入購物車</v-btn>
    </v-card-actions>
  </v-card>
</template>

<script setup>
import { useRouter } from 'vue-router'
import { useSnackbar } from 'vuetify-use-dialog'
import serviceUser from '@/services/user'
import { useUserStore } from '@/stores/user'

const createSnackbar = useSnackbar()
const user = useUserStore()
const router = useRouter()

const props = defineProps(['id', 'imageUrl', 'name', 'category', 'price', 'description'])
const formatedPrice = computed(() => new Intl.NumberFormat('zh-TW', { style: 'currency', currency: 'TWD', minimumFractionDigits: 0 }).format(props.price))

const addCart = async () => {
  try {
    if (!user.isLoggedIn) {
      router.push('/login')
      return
    }
    const { data } = await serviceUser.cart({ product: props.id, quantity: 1, replace: false })
    user.cart = data.result
    createSnackbar({
      text: '加入購物車成功',
      snackbarProps: {
        color: 'green',
      },
    })
  } catch (error) {
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

<style scoped lang="sass">
.v-card-text
  white-space: pre
  height: 100px
  overflow-y: hidden
</style>
