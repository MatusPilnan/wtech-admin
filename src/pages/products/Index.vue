<template>
  <q-page padding>
    <q-list highlight separator>
      <q-list-header>Zoznam produktov</q-list-header>
      <q-collapsible group="cat" v-for="category in categories" v-bind:key="category.name" :label="category.name">
        <q-collapsible separator group="subcat" v-for="subcategory in category.subcategories" v-bind:key="subcategory.name" :label="subcategory.name">
          <q-item dense highlight separator v-for="product in subcategory.products" v-bind:key="product.name">
            <q-item-side :image="getMiniImg(product)" />
            <q-item-main :label="product.name"  :sublabel="product.long_desc" :sublabel-lines="2">
              <q-item-tile>
                {{product.price}} €
              </q-item-tile>
            </q-item-main>
            <q-item-side right>
              <q-btn push color="warning" label="Upraviť" icon="edit" @click="$router.push({name: 'edit', params: {productID: product.id}})" />
              <q-btn push color="negative" label="Vymazať" icon="delete" @click="destroy(product.id, product.name)" />
            </q-item-side>
          </q-item>
        </q-collapsible>
      </q-collapsible>
    </q-list>
  </q-page>
</template>

<script>
import axios from 'axios'

export default {
  data: function () {
    return {
      manufacturers: [],
      categories: []
    }
  },
  // computed: {
  //   categories: {
  //     get () { return this.$store.state.moduleCatalog.catalog.categories }
  //   }
  // },
  methods: {
    getMiniImg: function (product) {
      var img = 'http://localhost:8000/img/logo.png'
      product.all_images.forEach(image => {
        if (image.mini === true) {
          img = 'http://localhost:8000/' + image.file
        }
      })
      return img
    },
    destroy (productID, productName) {
      this.$q.dialog({
        title: 'Vymazanie produktu',
        message: 'Naozaj chcete vymazat produkt ' + productName + '?',
        color: 'warning',
        ok: true,
        cancel: true
      }).then(() => {
        axios.delete(`http://localhost:8000/products/${productID}`)
          .then(r => {
            this.showNotification('Produkt uspesne vymazany', 'positive', productName)
            this.updateCatalog()
          })
          .catch(() => {
            this.showNotification('Produkt sa nepodarilo vymazat', 'negative', productName)
          })
      }).catch(() => {})
    },
    showNotification (message, color, detail) {
      this.$q.notify({
        message,
        timeout: 1000,
        color,
        detail,
        position: 'center'
      })
    },
    updateCatalog () {
      axios.get('http://localhost:8000/products/list')
        .then(response => {
          this.categories = response.data
        })
        .catch(e => {
          this.errors.push(e)
        })
    }
  },
  created () {
    this.updateCatalog()
  }
}
</script>
