<template>
  <q-page padding>
    <h1>Pridanie nového produktu</h1>
    <form>
      <q-select v-model="product.subcategory_id" :error="$v.product.subcategory_id.$error" float-label="Podkategória" :options="subcategoryOptions" />
      <q-field :count="255" >
        <q-input v-model="product.name" :error="$v.product.name.$error" float-label="Názov produktu"/>
      </q-field>
      <q-field  :count="255">
        <q-input v-model="product.short_desc" :error="$v.product.short_desc.$error" float-label="Krátky popis produktu"/>
      </q-field>
      <q-input v-model="product.long_desc" :error="$v.product.long_desc.$error" float-label="Podrobný opis produktu" type="textarea"/>
      <q-input v-model="product.price" :error="$v.product.price.$error" float-label="Cena produktu" type="number" :decimals="2" align="right" suffix=" EUR"/>
      <div class="row no-wrap">
        <q-select class="col-11" v-model="product.manufacturer_id" :error="$v.product.manufacturer_id.$error" float-label="Výrobca" :options="manufacturerOptions" />
        <q-btn @click="newManufacturer" class="col-1" flat icon="add" />
      </div>
      <!-- <q-input v-model="product.mini" type="file" />
      <q-input v-model="product.thumb" type="file" />
      <q-input v-model="product.imgs" type="file" /> -->
      <q-uploader ref="upMini" url="http://localhost:8000/images/upload" extensions=".jpg, .jpeg, .png" hide-upload-button float-label="Miniatúra v košíku" :additional-fields="addFieldsMini"/>
      <q-uploader ref="upThumb" url="http://localhost:8000/images/upload" extensions=".jpg, .jpeg, .png" hide-upload-button float-label="Náhľad v katalógu" :additional-fields="addFieldsThumb"/>
      <q-uploader ref="upImg" url="http://localhost:8000/images/upload" extensions=".jpg, .jpeg, .png" multiple hide-upload-button float-label="Obrázky" :additional-fields="addFields"/>
      <q-btn class="float-right q-mt-md" @click="save" label="Uložiť" color='primary' />
    </form>
    {{product.id}}
  </q-page>
</template>

<script>
import axios from 'axios'
import { required, maxLength } from 'vuelidate/lib/validators'

export default {
  data: function () {
    return {
      manufacturers: [],
      subcategories: [],
      product: {}
    }
  },
  computed: {
    manufacturerOptions: function () {
      var mo = []
      this.manufacturers.forEach(e => {
        mo.push({label: e.name, value: e.id})
      })
      return mo.sort((a, b) => {
        if (a.label < b.label) return -1
        if (a.label === b.label) return 0
        if (a.label > b.label) return 1
      })
    },
    subcategoryOptions: function () {
      var so = []
      this.subcategories.forEach(e => {
        so.push({label: e.name, value: e.id})
      })
      return so.sort((a, b) => {
        if (a.label < b.label) return -1
        if (a.label === b.label) return 0
        if (a.label > b.label) return 1
      })
    },
    addFieldsMini: function () {
      return [{name: 'product_id', value: this.productID}, {name: 'mini', value: true}]
    },
    addFieldsThumb: function () {
      return [{name: 'product_id', value: this.productID}, {name: 'thumbnail', value: true}]
    },
    addFields: function () {
      return [{name: 'product_id', value: this.productID}]
    }

  },
  props: {
    productID: {
      default: -1
    }
  },
  validations: {
    product: {
      name: { required, maxLength: maxLength(255) },
      short_desc: { required, maxLength: maxLength(255) },
      long_desc: { required },
      price: { required },
      manufacturer_id: { required },
      subcategory_id: { required }
    }
  },
  methods: {
    uploadMini: function () {
      return 'http://localhost:8000/images/upload/' + this.product.id
    },
    save: async function (e) {
      this.$v.product.$touch()

      if (this.$v.product.$error) {
        this.$q.notify({
          message: 'Skontrolujte zadane udaje',
          color: 'negative',
          timeout: 1001,
          position: 'center'
        })
        return
      }

      await axios.post('http://localhost:8000/products', this.product)
        .then(response => {
          this.productID = response.data.product_id
          this.$q.notify({
            message: 'Produkt uspesne ulozeny',
            color: 'positive',
            timeout: 1001,
            position: 'center'
          })
        })
        .catch(e => {
          this.$q.notify({
            message: 'Produkt sa nepodarilo ulozit',
            color: 'negative',
            timeout: 1000,
            position: 'center'
          })
        })
      this.$refs.upMini.upload()
      this.$refs.upThumb.upload()
      this.$refs.upImg.upload()
      this.$router.push('/')
    },
    loadManufacturers: function () {
      axios.get('http://localhost:8000/manufacturers/list')
        .then(response => {
          this.manufacturers = response.data
        })
        .catch(e => {
          this.errors.push(e)
        })
    },
    loadSubcategories: function () {
      axios.get('http://localhost:8000/subcategories/list')
        .then(response => {
          this.subcategories = response.data
        })
        .catch(e => {
          this.errors.push(e)
        })
    },
    newManufacturer: function () {
      this.$q.dialog({
        title: 'Nový výrobca',
        message: 'Zadajte meno nového výrobcu',
        prompt: {
          model: '',
          type: 'text'
        }
      }).then(data => {
        if (data === '') {
          return
        }
        console.log(data)
        axios.post('http://localhost:8000/manufacturers/new', {name: data})
          .then(response => {
            this.loadManufacturers()
          })
          .catch(e => {
            console.log(e)
          })
      }).catch(() => {})
    }
  },
  created () {
    this.loadSubcategories()
    this.loadManufacturers()
    if (this.productID !== -1) {
      axios.get('http://localhost:8000/product/' + this.productID + '/get')
        .then(response => {
          this.product = response.data
        })
        .catch(e => {
          this.errors.push(e)
        })
    }
  }
}
</script>

<style>
h1 {
  text-align: center;
  font-size: 2em;
  margin: 0
}
form {
  max-width: 500px;
  margin-left: auto;
  margin-right: auto
}
</style>
