<template>
  <div class="q-pa-lg row items-start q-gutter-md">
    <!-- Card -->
    <q-card class="my-card" :dark="true" v-for="item in allProducts.products" :key="allProducts.products.indexOf(item)">
      <q-card-section class="bg-blue-grey text-white">
        <div class="text-h6">
          {{item.productName}}
          <div class="absolute-right">
            <q-btn round dense flat icon="delete" color="dark" @click="deleteProduct(allProducts.products.indexOf(item))"/>
          </div>
        </div>
        <div class="text-subtitle2">{{item.productBrand}}</div>
        <q-linear-progress rounded style="height: 20px" :value="item.productTotalSaved/item.productPrice || 0" color="secondary" class="q-mt-sm">
          <div class="absolute-full flex flex-center">
            <q-badge color="secondary" text-color="white">{{item.productTotalSaved + '/' + item.productPrice}}</q-badge>
          </div>
        </q-linear-progress>
      </q-card-section>
        <q-card-actions align="center">
          <q-input outlined v-model="valueToAdd" label="Saved..." :dense="true" lazy-rules :rules="[ val => val !== null && val.trim() !== '' || 'Please type something', val => val >= 0 || 'Invalid Price']" :dark="true">
            <template v-slot:append>
              <q-btn round dense flat icon="add" @click="updateAquiredValue(allProducts.products.indexOf(item))" />
            </template>
          </q-input>
        </q-card-actions>
    </q-card>
    <!-- Modal Card -->
    <q-dialog v-model="modalOpen">
      <q-card style="width: 700px; max-width: 80vw;" :dark="true">
        <q-card-section>
          <div class="text-h6">Add new Product</div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          <q-form @submit="onSubmit" @reset="onReset" class="q-gutter-md">
            <q-input filled v-model="product.productName" label="Product Name..." lazy-rules :rules="[ val => val && val.trim().length > 0 || 'Please type something']" :dark="true"/>
            <q-input filled v-model="product.productBrand" label="Product Brand..." lazy-rules :rules="[ val => val && val.trim().length > 0 || 'Please type something']" :dark="true"/>
            <q-input filled v-model="product.productPrice" label="Product Price..." lazy-rules :rules="[ val => val !== null && val.trim() !== '' || 'Please type something', val => val >= 0 || 'Invalid Price']" :dark="true"/>
            <div>
              <q-btn label="Submit" type="submit" color="blue"/>
              <q-btn label="Reset" type="reset" color="blue" flat class="q-ma-sm" />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>
    <!-- Floating Button -->
    <q-page-sticky position="bottom-right" :offset="[18, 18]" @click.native="changeModalStatus">
      <q-btn fab icon="add" color="dark" />
    </q-page-sticky>
  </div>
</template>

<script>
export default {
  name: 'PageIndex',
  data: function () {
    return {
      modalOpen: false,
      product: {
        productName: null,
        productBrand: null,
        productPrice: null,
        productTotalSaved: 0
      },
      valueToAdd: null,
      allProducts: {
        products: []
      }
    }
  },
  methods: {
    changeModalStatus: function () {
      this.modalOpen = true
    },
    onReset: function () {
      this.product.productPrice = null
      this.product.productBrand = null
      this.product.productName = null
    },
    onSubmit: async function () {
      this.allProducts.products.push(this.product)
      await this.$q.localStorage.set('products', JSON.stringify(this.allProducts))
      this.modalOpen = false
      this.$forceUpdate()
    },
    getProducts: async function () {
      return await JSON.parse(this.$q.localStorage.getItem('products')).products || []
    },
    updateAquiredValue: async function (idProduct) {
      if (parseFloat(this.allProducts.products[idProduct].productTotalSaved) + parseFloat(this.valueToAdd) >= parseFloat(this.allProducts.products[idProduct].productPrice)) {
        this.allProducts.products[idProduct].productTotalSaved = parseFloat(this.allProducts.products[idProduct].productPrice)
      } else {
        this.allProducts.products[idProduct].productTotalSaved = parseFloat(this.allProducts.products[idProduct].productTotalSaved) + parseFloat(this.valueToAdd)
      }
      await this.$q.localStorage.set('products', JSON.stringify(this.allProducts))
      this.$forceUpdate()
    },
    deleteProduct: async function (idProduct) {
      this.allProducts.products = this.allProducts.products.slice(0, idProduct).concat(this.allProducts.products.slice(idProduct + 1, this.allProducts.products.length))
      await this.$q.localStorage.set('products', JSON.stringify(this.allProducts))
      this.$forceUpdate()
    }
  },
  mounted: function () {
    let selfContext = this
    this.getProducts().then(data => {
      selfContext.allProducts.products = data
    })
  }
}
</script>
