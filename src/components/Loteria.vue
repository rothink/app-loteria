<template>
  <v-container>
    <v-row justify="center">
      <v-col cols="12" md="8" lg="6">
        <v-card class="pa-6">
          <v-card-title class="text-h4 text-center mb-6">
            Sistema de Lotofácil
          </v-card-title>

          <v-card-subtitle class="text-center mb-6">
            Informe 10 números separados por espaço ou vírgula
          </v-card-subtitle>

          <v-form @submit.prevent="aoClicarGerar">
            <v-textarea
              v-model="entradaTexto"
              label="Números (ex: 01 04 06 07 10 13 19 22 24 25 ou 01,04,06,07,10,13,19,22,24,25)"
              variant="outlined"
              rows="3"
              class="mb-4"
              @input="processarEntrada"
            />

            <v-alert
              v-if="mensagemErro"
              type="error"
              density="compact"
              class="mb-4"
            >
              {{ mensagemErro }}
            </v-alert>

            <v-alert
              v-if="numerosProcessados.length === 10 && !mensagemErro"
              type="success"
              density="compact"
              class="mb-4"
            >
              {{ numerosProcessados.length }} números identificados
            </v-alert>

            <div v-if="numerosProcessados.length === 10 && !mensagemErro" class="mb-4">
              <div class="text-subtitle-2 mb-2">Números identificados:</div>
              <div class="d-flex flex-wrap">
                <v-chip
                  v-for="(numero, index) in numerosProcessados"
                  :key="index"
                  color="primary"
                  size="small"
                  class="ma-1"
                >
                  {{ numero.toString().padStart(2, "0") }}
                </v-chip>
              </div>
            </div>

            <v-btn
              type="submit"
              color="primary"
              size="large"
              block
              :disabled="!podeGerar"
            >
              Gerar Sugestões
            </v-btn>
          </v-form>

          <v-divider class="my-6" v-if="sugestoes.length > 0" />

          <div v-if="sugestoes.length > 0">
            <v-card-title class="text-h5 text-center mb-4">
              Números Sugeridos
            </v-card-title>
            <div class="text-center text-h6">
              {{ sugestoesFormatadas }}
            </div>
          </div>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, computed } from "vue";

const entradaTexto = ref("");
const numerosProcessados = ref([]);
const sugestoes = ref([]);
const mensagemErro = ref("");

const podeGerar = computed(() => {
  return numerosProcessados.value.length === 10;
});

const sugestoesFormatadas = computed(() => {
  return sugestoes.value.join(", ");
});

const processarEntrada = () => {
  mensagemErro.value = "";
  numerosProcessados.value = [];

  if (!entradaTexto.value.trim()) {
    return;
  }

  let texto = entradaTexto.value.trim();

  texto = texto.replace(/,/g, " ");

  const partes = texto.split(/\s+/).filter((p) => p.trim());

  const numeros = [];

  for (const parte of partes) {
    const numero = parseInt(parte.trim(), 10);

    if (isNaN(numero)) {
      continue;
    }

    if (numero < 1 || numero > 25) {
      mensagemErro.value = `Número ${numero} fora do intervalo válido (1-25)`;
      numerosProcessados.value = [];
      return;
    }

    if (numeros.includes(numero)) {
      mensagemErro.value = `Número ${numero} duplicado`;
      numerosProcessados.value = [];
      return;
    }

    numeros.push(numero);
  }

  if (numeros.length < 10) {
    mensagemErro.value = `Foram encontrados apenas ${numeros.length} números. Informe exatamente 10 números.`;
    numerosProcessados.value = [];
    return;
  }

  if (numeros.length > 10) {
    mensagemErro.value = `Foram encontrados ${numeros.length} números. Informe exatamente 10 números.`;
    numerosProcessados.value = [];
    return;
  }

  numerosProcessados.value = numeros.sort((a, b) => a - b);
};

const aoClicarGerar = () => {
  processarEntrada();
  if (podeGerar.value) {
    gerarSugestoes();
  }
};

const gerarSugestoes = () => {
  if (!podeGerar.value) return;

  const numerosInformados = new Set(numerosProcessados.value);
  const numerosDisponiveis = [];

  for (let i = 1; i <= 25; i++) {
    if (!numerosInformados.has(i)) {
      numerosDisponiveis.push(i);
    }
  }

  const sugestoesGeradas = [];
  const disponiveis = [...numerosDisponiveis];

  for (let i = 0; i < 6; i++) {
    const indiceAleatorio = Math.floor(Math.random() * disponiveis.length);
    sugestoesGeradas.push(disponiveis[indiceAleatorio]);
    disponiveis.splice(indiceAleatorio, 1);
  }

  sugestoes.value = sugestoesGeradas.sort((a, b) => a - b);
};
</script>
