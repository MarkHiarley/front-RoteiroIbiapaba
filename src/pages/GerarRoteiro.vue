<template>
<div 
  class="min-h-screen flex flex-col items-center justify-start bg-gray-100 p-6 bg-cover bg-center" :style="{ backgroundImage: `url(${bgForm})` }">    <div class="bg-white p-8 rounded-xl shadow-lg w-full max-w-2xl mb-8">
      <h1 class="text-3xl font-bold text-blue-700 text-center mb-8">Geração de Roteiro</h1>

      <form @submit.prevent="generateItinerary" class="space-y-6">

        <div class="bg-blue-100 text-blue-800 p-4 rounded-lg mb-6 text-sm w-full">
          <p><strong>Preencha os dados para gerar seu roteiro personalizado:</strong></p>
          <ul class="list-disc pl-5 mt-2">
            <li><strong>Cidade de Visita</strong>: digite o nome da cidade ou várias cidades separadas por "e" (ex: Tianguá, ou Tianguá e Ubajara). Se quiser gerar para toda a Serra da Ibiapaba, basta digitar <strong>Serra</strong> (obrigatório).</li>
            <li><strong>Dias de Viagem</strong>: quantos dias pretende viajar (obrigatório).</li>
            <li><strong>Interesses</strong>: seus interesses na viagem (opcional, ex: natureza, cultura, gastronômico).</li>
            <li><strong>Com Crianças</strong>: marque se for viajar com crianças.</li>
            <li><strong>Cidade de Hospedagem</strong>: selecione onde vai dormir (opcional, pode ser diferente da cidade de visita).</li>
          </ul>
        </div>


        <div>
          <label class="block text-gray-700 font-medium mb-2">Cidade de Visita</label>
          <input
            v-model="cidade"
            type="text"
            required
            placeholder="Ex: Viçosa do Ceará ou Tianguá e Ubajara"
            class="w-full rounded border border-gray-300 p-2 focus:ring-blue-500 focus:border-blue-500"
          />

        </div>

        <div>
          <label class="block text-gray-700 font-medium mb-2">Dias de Viagem</label>
          <input v-model.number="dias" type="number" min="1" required class="w-full rounded border border-gray-300 p-2 focus:ring-blue-500 focus:border-blue-500" />
        </div>

        <div>
          <label class="block text-gray-700 font-medium mb-2">Interesses</label>
          <input v-model="interesses" type="text" placeholder="Ex: natureza, cultura, aventura" class="w-full rounded border border-gray-300 p-2 focus:ring-blue-500 focus:border-blue-500" />
        </div>

        <div class="flex items-center space-x-3">
          <input v-model="comCriancas" type="checkbox" id="criancas" class="h-5 w-5 text-blue-600 focus:ring-blue-500 border-gray-300 rounded" />
          <label for="criancas" class="text-gray-700">Com crianças?</label>
        </div>

        <div>
          <label class="block text-gray-700 font-medium mb-2">Cidade de Hospedagem</label>
          <select v-model="hospedagem" class="w-full rounded border-gray-300 p-2 focus:ring-blue-500 focus:border-blue-500">
            <option value="">Selecione uma cidade (opcional)</option>
            <option value="Carnaubal">Carnaubal</option>
            <option value="Croatá">Croatá</option>
            <option value="Guaraciaba do Norte">Guaraciaba do Norte</option>
            <option value="Ibiapina">Ibiapina</option>
            <option value="Ipu">Ipu</option>
            <option value="São Benedito">São Benedito</option>
            <option value="Tianguá">Tianguá</option>
            <option value="Ubajara">Ubajara</option>
            <option value="Viçosa do Ceará">Viçosa do Ceará</option>
            <option value="Serra">Outra</option>
          </select>
        </div>

        <button type="submit" class="w-full bg-blue-600 hover:bg-blue-500 text-white py-3 rounded font-semibold transition">
          Gerar Roteiro
        </button>

        <p v-if="error" class="text-red-500 text-sm text-center mt-4">{{ error }}</p>
      </form>
    </div>

    <!-- Se carregando -->
    <div v-if="loading" class="flex flex-col items-center justify-center mt-10">
      <div class="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-blue-600"></div>
      <p class="mt-4 text-gray-700 font-semibold">Gerando seu roteiro, aguarde...</p>
    </div>

    <!-- Se roteiro gerado -->
    <div v-else-if="roteiro" class="bg-white p-8 rounded-xl shadow-lg w-full max-w-4xl prose prose-blue mt-8">
      <h2 class="text-2xl font-bold text-blue-700 mb-6 text-center">Seu Roteiro Personalizado</h2>

      <div class="flex justify-center mb-6">
        <button @click="downloadPDF" class="bg-green-600 hover:bg-green-500 text-white py-2 px-6 rounded-lg font-semibold transition">
          Baixar Roteiro em PDF
        </button>
      </div>

      <div id="roteiro-content">
        <p><strong>{{ roteiro.saudacao }}</strong></p>

        <div class="my-4">
          <h3 class="text-xl font-bold mb-2">Considerações Iniciais</h3>
          <ul class="list-disc pl-5">
            <li><strong>Clima:</strong> {{ roteiro.consideracoes_iniciais.clima }}</li>
            <li><strong>Roupas:</strong> {{ roteiro.consideracoes_iniciais.roupas }}</li>
            <li><strong>Transporte:</strong> {{ roteiro.consideracoes_iniciais.transporte }}</li>
            <li><strong>Alimentação:</strong> {{ roteiro.consideracoes_iniciais.alimentacao }}</li>
          </ul>
        </div>

        <div v-for="(dia, index) in roteiro.dias" :key="index" class="my-6">
          <h3 class="text-lg font-bold">{{ dia.dia }} - {{ dia.titulo }}</h3>
          <ul class="list-disc pl-5 mt-2">
            <li v-for="(atividade, idx) in dia.atividades" :key="idx">
              <strong>{{ atividade.hora }}</strong>: {{ atividade.descricao }}
            </li>
          </ul>
          <p class="mt-2"><strong>Resumo:</strong> {{ dia.resumo }}</p>
        </div>

        <div class="my-6">
          <h3 class="text-xl font-bold mb-2">Alternativas para Mau Tempo</h3>
          <ul class="list-disc pl-5">
            <li v-for="(item, idx) in roteiro.alternativas_mau_tempo" :key="idx">{{ item }}</li>
          </ul>
        </div>

        <div class="my-6">
          <h3 class="text-xl font-bold mb-2">Dicas Adicionais</h3>
          <ul class="list-disc pl-5">
            <li v-for="(item, idx) in roteiro.dicas_adicionais" :key="idx">{{ item }}</li>
          </ul>
        </div>

        <p class="text-center font-bold text-blue-600 mt-8">{{ roteiro.mensagem_final }}</p>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { apiPrivate } from '../services/api'
import html2pdf from 'html2pdf.js'
import bgForm from '../assets/fundo-formulario.png' // caminho correto
import { onBeforeMount } from 'vue'
import { useRouter } from 'vue-router'

const cidade = ref('')
const dias = ref(1)
const interesses = ref('')
const comCriancas = ref(false)
const hospedagem = ref('')
const error = ref('')
const roteiro = ref('')
const loading = ref(false)
const router = useRouter()

onBeforeMount(() => {
  const token = localStorage.getItem('access')
  if (!token) {
    window.location.href = '/login'
  }
})
const generateItinerary = async () => {
  try {
    loading.value = true
    error.value = ''
    roteiro.value = ''

    const token = localStorage.getItem('access')
    if (token) {
      apiPrivate.defaults.headers.common['Authorization'] = `Bearer ${token}` // 👈 corrigido aqui (era api errado)
    }

    const response = await apiPrivate.post('/roteiros/generate-itinerary/', {
      cidade: cidade.value,
      dias: dias.value,
      interesses: interesses.value.trim() !== '' ? interesses.value : undefined,
      com_criancas: comCriancas.value,
      hospedagem: hospedagem.value.trim() !== '' ? hospedagem.value : undefined
    })

    let rawText = response.data.roteiro

    rawText = rawText.trim()
    if (rawText.startsWith('```json')) {
      rawText = rawText.slice(7)
    }
    if (rawText.endsWith('```')) {
      rawText = rawText.slice(0, -3)
    }
    rawText = rawText.trim()

    roteiro.value = JSON.parse(rawText)

  } catch (err) {
    console.error('Erro na geração do roteiro:', err)
    if (err.response && err.response.status === 401) {
      error.value = 'Você precisa estar logado para gerar um roteiro.'
    } else {
      error.value = 'Erro ao gerar roteiro. Tente novamente.'
    }
  } finally {
    loading.value = false
  }
}

const downloadPDF = () => {
  const element = document.getElementById('roteiro-content')
  html2pdf()
    .from(element)
    .set({
      margin: 10,
      filename: 'roteiro_ibiapaba.pdf',
      html2canvas: { scale: 2 },
      jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
    })
    .save()
}
</script>



