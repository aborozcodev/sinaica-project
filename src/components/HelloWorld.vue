<template>
  <div class="position-fixed d-flex flex-column justify-content-center align-items-center" id="loader" v-if="state.isLoading">
    <div class="spinner"></div>
    <span>Cargando...</span>
  </div>
  <h1>Datos de Sinaica</h1>
  <p>{{ state.page}}</p>
  <select v-model="selectedDestination" @change="changeDestination">
    <option value="Selecionar destino">Selecionar destino</option>
    <option v-for="destination in destinations" :value="destination.nombre" :key="destination._id">{{destination.nombre}}</option>
  </select>

  <div v-if="sinaicaData.length > 0" class="container">
    <div class="row justify-content-center">

      <div class="col-12">
          <div class="pagination my-3">
            <button @click="prevPage" :disabled="state.currentPage == 1" class="btn btn-primary">Anterior</button>
            <p class="my-0">{{  state.currentPage }} de {{ state.totalPages }}</p>
            <button @click="nextPage" class="btn btn-primary">Siguiente</button>
          </div>
      </div>
      <div class="col-10">
        <table class="table table-striped table-hover w-100">
      <thead>
        <tr>
          <th>Estado</th>
          <th>Ciudad</th>
          <th>Valor</th>
          <th>Fecha</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="data in sinaicaData" :key="data._id">
          <td>{{ data.state}}</td>
          <td>{{ data.city }}</td>
          <td>{{ data.valororig }}</td>
          <td>{{ data.fecha }}</td>
        </tr>
      </tbody>
    </table>
    </div>
    </div>
    
    
    
  </div>

  <div v-else>
    <p>No se encontraron registros</p>
  </div>
  
</template>

<script setup>
import { defineProps, reactive,ref } from 'vue'

defineProps({
  msg: String
})

const state = reactive({ totalPages: 0, currentPage: 1,isLoading: false })
const selectedDestination = ref('Selecionar destino')

const destinations = ref([])
const sinaicaData = ref([])

fetch('https://api.datos.gob.mx/v2/sinaica-sistemas')
  .then(res => res.json())
  .then(data => {
      destinations.value = data.results
      state.totalPages = data.pagination.total
    }
    )


const getSinaicaData = async (destination, page) => {
  const res = await fetch(`https://api.datos.gob.mx/v2/sinaica?state=${destination}&page=${page}`)
  const data = await res.json()
  return data
}

const changeDestination = async () => {
  state.isLoading = true
  const data = await getSinaicaData(selectedDestination.value,state.currentPage)
  state.isLoading = false
  state.totalPages = Math.ceil(parseInt(data.pagination.total) / parseInt(data.pagination.pageSize))
  sinaicaData.value = data.results
  console.log(data)
}

const nextPage = () => {
  state.currentPage++
  changeDestination()
}

const prevPage = () => {
  state.currentPage--
  changeDestination()
}

</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.pagination{
  display: flex;
  align-items: center;
  justify-content: center;
}

.pagination p {
  margin-left: 15px;
  margin-right: 15px;
}

table{
  width: 35%;
}

#loader{
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0,0,0,.2);
}

.spinner {
    border: 4px solid rgba(0,0,0,0.6);
    width: 60px;
    height: 60px;
    border-radius: 50%;
    border-left-color: transparent;
    margin: 0 auto;
    animation: spin 1s linear infinite;
}

@keyframes spin {
    0% {
        transform: rotate(0deg);
    }

    100% {
        transform: rotate(360deg);
    }
}
</style>