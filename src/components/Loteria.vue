<template>
  <v-container>
    <v-row justify="center">
      <v-col cols="12" md="8" lg="6">
        <v-card class="pa-6">
          <v-card-title class="text-h4 text-center mb-6">
            Sistema de Loteria
          </v-card-title>

          <v-card-subtitle class="text-center mb-6">
            Informe 10 números e receba 6 sugestões diferentes
          </v-card-subtitle>

          <v-form @submit.prevent="gerarSugestoes">
            <v-row>
              <v-col
                v-for="(numero, index) in numeros"
                :key="index"
                cols="6"
                sm="4"
                md="3"
              >
                <v-text-field
                  v-model.number="numeros[index]"
                  :label="`Número ${index + 1}`"
                  type="number"
                  min="1"
                  max="60"
                  variant="outlined"
                  density="compact"
                  @input="validarNumero(index)"
                />
              </v-col>
            </v-row>

            <v-btn
              type="submit"
              color="primary"
              size="large"
              block
              class="mt-4"
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
            <v-row justify="center">
              <v-col
                v-for="(sugestao, index) in sugestoes"
                :key="index"
                cols="auto"
              >
                <v-chip color="purple" size="large" class="ma-1">
                  {{ sugestao }}
                </v-chip>
              </v-col>
            </v-row>
          </div>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, computed } from "vue";

const numeros = ref([...Array(10)].map(() => null));
const sugestoes = ref([]);

const podeGerar = computed(() => {
  const numerosValidos = numeros.value.filter(
    (n) => n !== null && n >= 1 && n <= 60
  );
  const numerosUnicos = new Set(numerosValidos);
  return numerosValidos.length === 10 && numerosUnicos.size === 10;
});

const validarNumero = (index) => {
  const valor = numeros.value[index];
  if (valor !== null) {
    if (valor < 1) {
      numeros.value[index] = 1;
    } else if (valor > 60) {
      numeros.value[index] = 60;
    }
  }
};

const gerarSugestoes = () => {
  if (!podeGerar.value) return;

  const numerosInformados = new Set(numeros.value);
  const numerosDisponiveis = [];

  for (let i = 1; i <= 60; i++) {
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
