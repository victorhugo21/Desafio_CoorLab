<template>
  <div class="title max-height">
    <b-navbar toggleable="lg" type="dark" variant="info">
      <b-navbar-brand class="ml-2">
        <b>{{ appName }}</b>
      </b-navbar-brand>
    </b-navbar>
    <div class="container column d-flex justify-content-center flex-column ">
      <div class="col-md-8 card shadow mx-auto text-dark">
        <div class="card-title bg-info d-flex align-items-center"> <b-icon class="mx-5 my-3" icon="truck"
            font-scale="2" />{{ appName }}</div>
        <div class="card-body d-flex justify-content-between">
          <FormBestTransport :parametro="destinos" v-on:on-Submit="analiza" />
          <div class="mx-4" v-if="resultado">
            <p class="mx-3 my-3">Estas são as melhores alternativas de frete que encontramos para você.</p>
            <div class="d-flex justify-content-between">
              <div class="card mb-3" style="max-width: 540px;">
                <div class="row g-0">
                  <div class="rounded-left col-md-4 bg-info  d-flex justify-content-center align-items-center ">
                    <b-icon class="" icon="cash-coin" font-scale="2" />
                  </div>
                  <div class="col-md-8">
                    <div class="card-body">
                      <h5 class="card-title">Frete com menor valor</h5>
                      <p class="card-text">Transportadora: {{ resultado.transporteMaisBarato.name }}</p>
                      <p class="card-text">Tempo estimado: {{ resultado.transporteMaisBarato.lead_time }}
                        <!-- <small class="text-muted">Last updated 3 mins ago</small> -->
                      </p>
                    </div>
                  </div>
                </div>
              </div>
              <div class="card mb-3">
                <div class="card-body">
                  <h5 class="card-title">Preço</h5>
                  <p class="card-text">R$ {{ resultado.transporteMaisBarato.cost }}</p>
                </div>
              </div>
            </div>
            <div class="d-flex justify-content-between">
              <div class="card mb-3" style="max-width: 540px;">
                <div class="row g-0">
                  <div class="rounded-left col-md-4 bg-info  d-flex justify-content-center align-items-center ">
                    <b-icon class="" icon="stopwatch" font-scale="2" />
                  </div>
                  <div class="col-md-8">
                    <div class="card-body">
                      <h5 class="card-title">Frete mais rápido</h5>
                      <p class="card-text">Transportadora: {{ resultado.transporteMaisRapido.name }}</p>
                      <p class="card-text">Tempo estimado: {{ resultado.transporteMaisRapido.lead_time }}</p>
                    </div>
                  </div>
                </div>
              </div>
              <div class="card mb-3">
                <div class="card-body">
                  <h5 class="card-title">Preço</h5>
                  <p class="card-text">R$ {{ resultado.transporteMaisRapido.cost }}</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {
  BNavbar,
  BNavbarBrand,
} from 'bootstrap-vue'

import FormBestTransport from './FormBestTransport.vue'
export default {
  components: {
    BNavbar,
    BNavbarBrand,
    FormBestTransport,
  },
  data() {
    const appName = ''

    return {
      appName,
      cotacoes: [],
      destinos: [{ text: 'Selecione o destino', value: null }],
      peso: null,
      destino: '',
      resultado: null,
    }
  },
  created() {
    // Implemente aqui o GET dos dados da API REST
    // para que isso ocorra na inicialização da pagina
    const apiUrl = "http://localhost:3000/transport";

    // Realiza a requisição GET para a API REST
    fetch(apiUrl)
      .then((response) => response.json())
      .then((data) => {
        this.cotacoes = data;
        data.forEach(element => {
          if (!this.destinos.includes(element.city))
            this.destinos.push(element.city);
        });
      })
      .catch((error) => {
        console.error(error);
        //alert("Erro ao obter as cotações de frete.");
      });

    this.appName = 'Melhor Frete'
  },

  methods: {
    // Implemente aqui os metodos utilizados na pagina
    analiza(form) {
      this.destino = form.destino;
      this.peso = form.peso;

      const dadoFiltrado = this.cotacoes.filter(
        (item) => item.city.toLowerCase() === this.destino.toLowerCase());
      if (dadoFiltrado.length > 0) {
        const transporteMaisBarato = this.getTransporteMaisBarato(dadoFiltrado);
        const transporteMaisRapido = this.getTransporteMaisRapido(dadoFiltrado);

        this.resultado = {
          transporteMaisBarato,
          transporteMaisRapido,
        };
      } else {
        this.resultado = null;
        alert("Nenhuma cotação encontrada para a cidade de destino.");
      }
    },
    getTransporteMaisBarato(data) {
      const sortedData = data.sort(
        (a, b) =>
          this.calculaCustoTotal(a) - this.calculaCustoTotal(b)
      );
      
      const cheapest = sortedData[0];
      console.log(this.calculaCustoTotal(cheapest));
      return {
        name: cheapest.name,
        cost: (this.calculaCustoTotal(cheapest) * this.peso).toFixed(2),
        lead_time: cheapest.lead_time,
      };
    },
    getTransporteMaisRapido(data) {
      const sortedData = data.sort(
        (a, b) =>
          parseInt(a.lead_time.match(/\d+/)[0])-
          parseInt(b.lead_time.match(/\d+/)[0])
      );
      const fastest = sortedData[0];

      return {
        name: fastest.name,
        cost: (this.calculaCustoTotal(fastest) * this.peso).toFixed(2),
        lead_time: fastest.lead_time,
      };
    },
    calculaCustoTotal(transport) {
      const costLight = parseFloat(transport.cost_transport_light.replace("R$ ", ""));
      const costHeavy = parseFloat(transport.cost_transport_heavy.replace("R$ ", ""));

      if (this.peso <= 100) {
        return costLight;
      } else {
        return costHeavy;
      }
    },
  },
}
</script>

<style scoped>
.title .navbar {
  background-color: #00aca6 !important;
}

.title .navbar-brand {
  margin-left: 20px;
}

.container {
  height: 80vh;
}

.card {
  border-radius: 15px;
}

.card-title {
  border-top-left-radius: 15px;
  border-top-right-radius: 15px;
}
</style>
