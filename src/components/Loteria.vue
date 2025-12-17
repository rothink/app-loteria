<template>
  <v-container>
    <v-row justify="center">
      <v-col cols="12" md="8" lg="6">
        <v-card class="pa-6">
          <div class="d-flex justify-end mb-2">
            <v-btn
              :icon="theme.global.name.value === 'dark' ? 'mdi-weather-sunny' : 'mdi-weather-night'"
              variant="text"
              @click="toggleTheme"
            />
          </div>
          <v-card-title class="text-h4 text-center mb-6">
            Sistema de Jogatina semanal
          </v-card-title>



          <div class="d-flex justify-end mb-2">
            <v-btn
              variant="text"
              size="small"
              @click="camposDesbloqueados = !camposDesbloqueados"
            >
              <v-icon start>
                {{ camposDesbloqueados ? 'mdi-lock' : 'mdi-lock-open-variant' }}
              </v-icon>
              {{ camposDesbloqueados ? 'Bloquear' : 'Desbloquear' }} Configurações
            </v-btn>
          </div>



          <v-row class="mb-4">
            <v-col cols="6">
              <v-text-field
                v-model.number="quantidadeEntrada"
                label="Quantidade de entrada"
                type="number"
                min="1"
                max="25"
                variant="outlined"
                density="compact"
                :disabled="!camposDesbloqueados"
              />
            </v-col>
            <v-col cols="6">
              <v-text-field
                v-model.number="quantidadeSaida"
                label="Quantidade de saída"
                type="number"
                min="1"
                max="25"
                variant="outlined"
                density="compact"
                :disabled="!camposDesbloqueados"
              />
            </v-col>
          </v-row>

          <v-card-subtitle class="text-center mb-4">
            Informe números separados por espaço ou vírgula
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
              v-if="numerosProcessados.length === quantidadeEntrada && !mensagemErro"
              type="success"
              density="compact"
              class="mb-4"
            >
              {{ numerosProcessados.length }} números identificados
            </v-alert>

            <div v-if="numerosProcessados.length === quantidadeEntrada && !mensagemErro" class="mb-4">
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
              :disabled="!podeGerar || carregando"
              :loading="carregando"
            >
              Gerar Sugestões
            </v-btn>
          </v-form>

          <v-overlay v-model="carregando" class="align-center justify-center">
            <v-card class="pa-6 text-center" min-width="300">
              <v-progress-circular
                indeterminate
                color="primary"
                size="64"
                class="mb-4"
              />
              <div class="text-h6 mb-2">Gerando números...</div>
              <div class="text-body-2 text-medium-emphasis">
                Executando {{ quantidadeGeracoes }} gerações aleatórias
              </div>
            </v-card>
          </v-overlay>

          <v-divider class="my-6" v-if="sugestoes.length > 0" />

          <div v-if="sugestoes.length > 0">
            <v-card-title class="text-h5 text-center mb-2">
              Números Sugeridos
            </v-card-title>
            <div class="text-center text-subtitle-1 mb-4">
              Quantidade: {{ sugestoes.length }} números
            </div>
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
import { useTheme } from "vuetify";

const theme = useTheme();

const toggleTheme = () => {
  theme.global.name.value = theme.global.name.value === "dark" ? "light" : "dark";
};

const entradaTexto = ref("");
const numerosProcessados = ref([]);
const sugestoes = ref([]);
const mensagemErro = ref("");
const carregando = ref(false);
const quantidadeGeracoes = ref(0);
const quantidadeEntrada = ref(10);
const quantidadeSaida = ref(6);
const camposDesbloqueados = ref(false);

const podeGerar = computed(() => {
  return numerosProcessados.value.length === quantidadeEntrada.value;
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

  if (numeros.length < quantidadeEntrada.value) {
    mensagemErro.value = `Foram encontrados apenas ${numeros.length} números. Informe exatamente ${quantidadeEntrada.value} números.`;
    numerosProcessados.value = [];
    return;
  }

  if (numeros.length > quantidadeEntrada.value) {
    mensagemErro.value = `Foram encontrados ${numeros.length} números. Informe exatamente ${quantidadeEntrada.value} números.`;
    numerosProcessados.value = [];
    return;
  }

  numerosProcessados.value = numeros.sort((a, b) => a - b);
};

const aoClicarGerar = async () => {
  processarEntrada();
  if (podeGerar.value) {
    await gerarSugestoes();
  }
};

const gerarSugestoes = async () => {
  if (!podeGerar.value) return;

  carregando.value = true;
  sugestoes.value = [];

  const numerosInformados = new Set(numerosProcessados.value);
  const numerosDisponiveis = [];

  for (let i = 1; i <= 25; i++) {
    if (!numerosInformados.has(i)) {
      numerosDisponiveis.push(i);
    }
  }

  const quantidadeAleatoria = Math.floor(Math.random() * 40) + 1;
  quantidadeGeracoes.value = quantidadeAleatoria;

  const inicio = Date.now();
  const duracao = 5000;
  const intervalo = duracao / quantidadeAleatoria;

  for (let geracao = 0; geracao < quantidadeAleatoria; geracao++) {
    const sugestoesGeradas = [];
    const disponiveis = [...numerosDisponiveis];

    for (let i = 0; i < quantidadeSaida.value; i++) {
      if (disponiveis.length === 0) break;
      const indiceAleatorio = Math.floor(Math.random() * disponiveis.length);
      sugestoesGeradas.push(disponiveis[indiceAleatorio]);
      disponiveis.splice(indiceAleatorio, 1);
    }

    sugestoes.value = sugestoesGeradas.sort((a, b) => a - b);

    if (geracao < quantidadeAleatoria - 1) {
      const tempoDecorrido = Date.now() - inicio;
      const proximaGeracao = (geracao + 1) * intervalo;
      const tempoEspera = Math.max(0, proximaGeracao - tempoDecorrido);
      await new Promise((resolve) => setTimeout(resolve, tempoEspera));
    }
  }

  const tempoDecorrido = Date.now() - inicio;
  const tempoRestante = duracao - tempoDecorrido;
  if (tempoRestante > 0) {
    await new Promise((resolve) => setTimeout(resolve, tempoRestante));
  }

  carregando.value = false;
};
</script>
