<template>
  <div class="container">
    <h1 class="title mt-20">Actualizar productos de MercadoLibre ({{ meliTotal }})</h1>
    <h2
      class="subtitle"
    >Aplicá las configuraciones de publicaciones que te gustaría que tus productos cumplan</h2>
    <hr />
    <button
      @click="updateEach"
      class="button is-success is-fullwidth"
    >MODIFICAR PRODUCTOS A MERCADOLIBRE</button>
    <hr />
    Resultado ({{ total }})
    <div class="row">
      <div class="col-md-12">
        <table class="table is-narrow is-hoverable is-fullwidth">
          <thead>
            <th>Estado</th>
            <th>Producto</th>
            <th>Mensaje</th>
          </thead>
          <tbody>
            <tr v-for="response_ in response">
              <td>
                <div
                  v-bind:class=" response_.status == 200 ? 'button is-success ' : 'button is-danger ' "
                >
                  <span class v-if="response_.status == 200">APROBADO</span>
                  <span v-else>NO APROBADO</span>
                </div>
              </td>
              <td>{{ response_.title }}</td>
              <td>
                <div v-if="response_.status == 200">
                  <b>Tipo de publicación:</b>
                  {{ response_.type }}
                  <br />
                  <b>Código MercadoLibre:</b>
                  {{ response_.code }}
                  <br />
                  <b>Precio MercadoLibre:</b>
                  {{ response_.price }}
                  <br />
                  <b>Porcentaje de recargo aplicado:</b>
                  {{ response_.percent }}
                </div>
                <div v-else>{{ response_.error.message }}</div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      products: [],
      response: [],
      total: 0,
      isLoading: false,
      meliTotal:0
    };
  },
  async mounted() {
    const products = await axios.get(
      process.env.apiUrl + "/product",
      this.$cookies.get("header-token")
    );
    this.products = products.data;
    await this.$data.products.forEach(async product => {
      await product.mercadolibre.forEach(async meli => {
        this.meliTotal += 1;
      });
    });
  },
  methods: {
    updateEach: async function() { 
      this.isLoading = true;
      this.response = [];
      this.total = 0;
      var checkToken = await axios.get(
        process.env.apiUrl + "/mercadolibre/refresh-token"
      );
      await this.$data.products.forEach(async product => {
        await product.mercadolibre.forEach(async meli => {
          const meliPostItem = await axios.put(
            process.env.apiUrl + "/mercadolibre/item/" + meli.code,
            {
              item: product,
              shipping: meli.shipping,
              percent: meli.percent,
              type: meli.type
            },
            this.$cookies.get("header-token")
          );
          if (meliPostItem.data.status === 200) {
            this.$data.total += 1;
          }
          await this.$data.response.push(meliPostItem.data);
        });
      });
      this.isLoading = false;
    }
  }
};
</script> 