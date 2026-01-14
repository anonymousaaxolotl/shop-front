<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-text-field v-model="search.text" hide-details></v-text-field>
      </v-col>
      <v-col cols="12">
        <v-chip-group v-model="search.categories" multiple selected-class="text-primary">
          <v-chip
            v-for="option in categoryOptions"
            :key="option"
            filter
            :text="option"
            :value="option"
          ></v-chip>
        </v-chip-group>
      </v-col>
      <v-col cols="12">
        <v-select
          v-model="search.sort"
          hide-details
          item-title="text"
          :items="sortOptions"
          return-object
        ></v-select>
      </v-col>
      <v-col cols="12">
        <v-data-iterator :items="filteredProducts" :items-per-page="12" :page="page">
          <template #default="{ items }">
            <v-row class="my-3">
              <v-col
                v-for="item in items"
                :key="item._id"
                cols="12"
                lg="3"
                md="6"
              >
                <card-product v-bind="item.raw"></card-product>
              </v-col>
            </v-row>
          </template>
          <template #footer="{ pageCount }">
            <v-pagination v-model="page" :length="pageCount"></v-pagination>
          </template>
        </v-data-iterator>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { computed, ref } from 'vue'
import { useSnackbar } from 'vuetify-use-dialog'
import serviceProduct from '@/services/product'

const createSnackbar = useSnackbar()

const search = ref({
  text: '',
  categories: [],
  sort: {
    text: '名稱',
    key: 'name',
    direction: 1,
  },
})

const products = ref([])
const filteredProducts = computed(() => {
  return products.value.filter(product => {
    const match = product.name.toLowerCase().includes(search.value.text.toLowerCase())
    const categoryMatch = search.value.categories.length === 0 || search.value.categories.includes(product.category)
    return match && categoryMatch
  }).toSorted((a, b) => {
    // return 0 順序不變
    // return < 0  a 在前
    // return > 0  b 在前
    const direction = search.value.sort.direction
    const key = search.value.sort.key
    // 若 key 為 price，假設
    // a.price = 80
    // b.price = 100
    // direction: 1 時小到大
    // 80 > 100 --> -1 * 1 --> -1 --> -1 <0 --> a 在前 --> 80 在前正序
    // direction: -1 時大到小
    // 80 > 100 --> -1 * -1 ---> 1 ---> 1 > 0 ---> b 在前 --> 100 在前倒序
    return a[key] > b[key] ? 1 * direction : -1 * direction
  })
})
const page = ref(1)

const categoryOptions = ['手機', '電腦', '平板', '玩具', '食品', '衣服', '遊戲', '書籍', '其他']
const sortOptions = [
  { text: '名稱', key: 'name', direction: 1 },
  { text: '價格: 低到高', key: 'price', direction: 1 },
  { text: '價格: 高到低', key: 'price', direction: -1 },
  { text: '新到舊', key: 'createdAt', direction: -1 },
  { text: '舊到新', key: 'createdAt', direction: 1 },
]

const getProducts = async () => {
  try {
    const { data } = await serviceProduct.get()
    products.value = data.result
  } catch {
    createSnackbar({
      text: '發生錯誤',
      snackbarProps: {
        color: 'red',
      },
    })
  }
}
getProducts()
</script>

<route lang="yaml">
meta:
  title: 首頁
</route>
