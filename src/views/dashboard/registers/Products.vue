<template>
    <div id="dashboard-warehouse" class="columns">
        <div id="products-list" class="column" style="padding-left:2rem">
          <ul v-for="(index, i) in products" :key="i">

            <li>
            <div class="card" >
              <div class="card-content" >
                <label class="title">{{index.name}}</label><br>
                <label class="subtitle">Id</label> <label class="label">{{index.id}}</label>
                <label class="subtitle">Nome</label> <label class="label">{{index.name}}</label>
                <label class="subtitle">Quantidade</label> <label class="label">{{index.quantity}} </label>
                <label class="subtitle">Descrição</label> <label class="label">{{index.description}}</label>
                <label class="subtitle">Tipo</label> <label class="label">{{index.type}}</label>
                <label class="subtitle">Categoria</label> <label class="label">{{index.category}}</label>
                <label class="subtitle">Unidade de Medida</label> <label class="label">{{index.unit_measurement}}</label>
                <label class="subtitle">Data de Validade</label> <label class="label">{{index.expiration_date}}</label>
                <div class="buttons">
                  <input class="button is-danger" type="button" value="Excluir" @click.prevent="onDelete(index.id)">
                  <input class="button is-warning" type="button" value="Editar" @click.prevent="onEdit(index.id)">
                </div>
              </div>
            </div>
            </li>

          </ul>
        </div>

        <div id="products-register-form" class="column" style="padding-left:2rem; padding-right:3rem">
           <label v-if="product.edited === false" class="title" >Cadastre um Novo Produto</label>
           <label v-else class="title" >Editar um Produto</label>

          <div class="field">
            <label class="label">Nome do Produto</label>
              <div class="control">
                <input class="input is-rounded is-success" type="text" placeholder="Nome" v-model="product.name">
              </div>
          </div>

          <div class="field">
            <label class="label">Quantidade</label>
              <div class="control">
                <input class="input is-rounded" type="number" placeholder="Quantidade" v-model="product.quantity">
              </div>
          </div>
          
          <div class="field">
            <label class="label">Descrição</label>
              <div class="control">
                <textarea class="textarea is-small" placeholder="Descrição" rows="5" v-model="product.description"></textarea>
              </div>
          </div>

          <div class="field">
            <label class="label">Tipo</label>
              <div class="control">
                <select class="select" v-model="product.type">
                  <option value="">Selecione um Tipo</option>
                  <option value="Liquido">Líquido</option>
                  <option value="Solido">Sólido</option>
                  <option value="Pasta">Pasta</option>
                  <option value="Gel">Gel</option>
                </select>
              </div>
          </div>

          <div class="field">
            <label class="label">Categoria</label>
              <div class="control">
                <select class="select" v-model="product.category">
                  <option value="">Selecione uma Categoria</option>
                  <option value="Limpeza">Limpeza</option>
                  <option value="Equipamento">Equipamento</option>
                  <option value="EPI">EPI</option>
                  <option value="Patrimonio">Patrimônio</option>
                  <option value="Outros">Outros</option>
                </select>
              </div>
          </div>

           <div class="field">
            <label class="label">Unidade de Medida</label>
              <div class="control">
                <select class="select" v-model="product.unit_measurement">
                  <option value="">Selecione uma Unidade de Medida</option>
                  <option value="Unidade">Unidade</option>
                  <option value="Litro">Litro</option>
                  <option value="Caixa">Caixa</option>
                  <option value="Bomba">Bomba (5 Litros)</option>
                  <option value="Pacote">Pacote</option>
                  <option value="Fardo">Fardo</option>
                </select>
              </div>
          </div>

          
          <div class="field">
            <label class="label">Data de Validade</label>
              <div class="control">
                <input class="input is-rounded" pattern="dd/MM/yyyy" type="date" placeholder="Data de Validade" v-model="product.expiration_date">
              </div>
          </div>

          <div class="buttons">
            <input v-if="product.edited === false" class="button is-link is-light is-rounded" type="button" value="Criar" @click.prevent="onSave">
            <input v-else class="button is-link is-light is-rounded" type="button" value="Editar" @click.prevent="onSave">
          </div>

        </div>
    </div>
</template>
<script>
import { onMounted, ref } from 'vue'
import router from '../../../router'
const { httpClient } = require('../../../core/application/outside/http_client_config')

export default {
  name: 'Register_Products',
  setup() {
    const products = ref([])
    const product = ref({
      name:"",
      quantity:0,
      description:"",
      type:"",
      category:"",
      unit_measurement:"",
      expiration_date:"",
      active:true,
      edited:false
    })

    const formatDate = (str) => {
      return str.split('-').reverse().join('-');
    }

    const loadProducts = async () => {
      try{
        const response = await httpClient.get('/products/findallactive', {
          headers: {'Authorization': 'Bearer ' + localStorage.getItem('token')}
        })
        products.value = response.data
        console.log(products.value)
      }catch(e){
        router.push({name:'Login'})
        console.error(e)
      }
    }

    const onSave = async () => {
      try {
        if(product.value.edited === false){
          console.log(product.value)
          const response = await httpClient.post('/products/create', product.value, {
            headers: {'Authorization': 'Bearer ' + localStorage.getItem('token')}
          })
          console.log(response.data)
        }else{
          let product_id = localStorage.getItem('product_id')
          const response = await httpClient.put(`/products/updatebyid/${product_id}`, product.value, {
             headers: {'Authorization': 'Bearer ' + localStorage.getItem('token')}
          })
          console.log(response.data)
          console.log('edited')
        }
      } catch (e) {
        console.error(e)
      }
    }

    const onDelete = async ( _id ) => {
      try{
        const response = await httpClient.delete(`/products/deletebyid/${ _id }`, {
          headers: {'Authorization': 'Bearer ' + localStorage.getItem('token')}
        })
        console.log(response.data)
      }catch(e){
        console.error(e)
      }
    }

    const onEdit = async ( _id ) => {
      try{
        const response = await httpClient.get(`/products/findbyid/${ _id }`, {
          headers: {'Authorization': 'Bearer ' + localStorage.getItem('token')}
        })
        console.log(response.data)
        product.value = response.data
        product.value.edited = true
        product.value.expiration_date = formatDate(response.data.expiration_date)
        console.log(product.value.expiration_date)
        localStorage.setItem('product_id', response.data.id)
      }catch(e){
        console.error(e)
      }
    }

    onMounted(async ()=>{
      await loadProducts()
    })

    return {
      product,
      products,
      onSave,
      onDelete,
      onEdit
    }
  },
  created() {
    if(localStorage.getItem('token') == null || localStorage.getItem('token') == undefined){
      router.push({name:'Login'})
    }
  },
  
}
</script>