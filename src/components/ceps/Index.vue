<template>
  <div class="fluid">
    <div class="col-sm-4">
      <p></p>
      <b-card-group deck class="mb-4">
        <b-card border-variant="light" header="CEP" class="text-center">
          <b-card-text>
            <!-- Formulario Create e add CEP -->
            <b-form @submit.prevent>
              <label for="feedbackCep">Localize o CEP para Adicionar a Lista</label>
              <b-input
                autofocus
                :sort-by.sync="sortBy"
                :sort-desc.sync="sortDesc"
                type="text"
                v-model="opcoes.cep"
                maxlength="8"
                v-on:keyup="getCep"
                id="feedbackCep"
              />
              <b-form-invalid-feedback :state="opcoes.error">
                <span class="font-weight-bold">{{msg.error}}</span>
              </b-form-invalid-feedback>
              <b-form-valid-feedback :state="opcoes.error">{{msg.sucess}}</b-form-valid-feedback>
              <p/>
              <div
                class="alert alert-primary"
                v-bind:class="{ 'alert alert-danger': opcoes.error  }"
                role="alert"
              >{{msg}}</div>
            </b-form>
          </b-card-text>
        </b-card>
      </b-card-group>
    </div>

    <div class="col-sm-12">
      <b-table bordered responsive striped hover :items="items" table-id :fields="fields">
        <b-container slot="acoes" class="col-md-4" slot-scope="row">
          <b-row>
            <i
              v-b-popover.hover.top="'Editar CEP'"
              @click="cepEdit(row.item, row.index)"
              class="btn material-icons text-info cursor-pointer"
            >
              <b-button variant="btn btn-sm btn-primary rounded-circle material-icons">edit</b-button>
            </i>

            <i
              v-b-popover.hover.top="'Remover CEP'"
              @click="cepRemover(row.index)"
              class="btn material-icons text-info cursor-pointer"
            >
              <b-button variant="btn btn-sm btn-danger rounded-circle material-icons">delete_forever</b-button>
            </i>
          </b-row>
        </b-container>
      </b-table>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import lodash from "lodash";
const axios = require("axios");

export default {
  name: "IndexCep",
  components: {},
  data() {
    return {
      status: false,
      msg: "O CEP deve conter 8 Caracteres.",
      sortBy: "cep",
      sortDesc: false,
      opcoes: {
        item: {},
        index: 0,
        cep: "041040",
        create: true,
        edit: false,
        remove: false,
        error: true
      },

      fields: {
        cep: {
          label: "CEP",
          sortable: true
        },
        logradouro: {
          label: "Logradouro",
          sortable: false
        },
        complemento: {
          label: "Complemento",
          sortable: false
        },
        acoes: {
          label: "Opções",
          class: "text-center"
        }
      },
      items: []
    };
  },
  methods: {
    cepEdit(item, index) {
      this.msg =
        "O CEP Selecionado: " +
        lodash.replace(item.cep, "-", "") +
        ", localize outro CEP para substituílo na lista";

      this.opcoes = {
        error: false,
        index: index,
        item: item,
        cep: lodash.replace(item.cep, "-", ""),
        create: false,
        edit: true,
        remove: false
      };
    },

    cepRemover(index) {
      this.msg = "O CEP deve conter 8 Caracteres.";
      this.opcoes = {
        error: true,
        cep: "041040"
      };
      this.items.splice(index, 1);
    },

    getCep() {
      if (this.opcoes.cep.length == 8 && this.opcoes.edit == true) {
        axios
          .get("https://viacep.com.br/ws/" + this.opcoes.cep + "/json/")
          .then(response => {
            if (response.data.erro) {
              this.msg =
                "CEP Pesquisado: " + this.opcoes.cep + " não foi localizado";
              this.opcoes.error = true;
            } else {
              this.msg =
                "CEP Pesquisado: " +
                this.opcoes.cep +
                " foi localizado e adicionado a sua lista";
              this.opcoes.item = {
                cep: response.data.cep,
                logradouro: response.data.logradouro,
                complemento: response.complemento,
                bairro: response.data.bairro,
                localidade: response.data.localidade,
                uf: response.data.uf
              };

              this.opcoes.error = false;
              this.items.splice(this.opcoes.index, 1);
              this.items.push(this.opcoes.item);
              this.opcoes.edit = false;
              this.opcoes.create = true;
              lodash.orderBy(this.items, ['cep'], ['asc']);
            }
          })
          .catch(erro => {
            this.msg = "Error ao fazer requisição na API VIACEP";
            this.opcoes.error = true;
          });
      }

      if (this.opcoes.cep.length == 8 && this.opcoes.create == true) {
        axios
          .get("https://viacep.com.br/ws/" + this.opcoes.cep + "/json/")
          .then(response => {
            if (response.data.erro) {
              this.msg =
                "CEP Pesquisado: " + this.opcoes.cep + " não foi localizado";
              this.opcoes.error = true;
            } else {
              this.msg =
                "CEP Pesquisado: " +
                this.opcoes.cep +
                " foi localizado e adicionado a sua lista";
              this.opcoes.item = {
                cep: response.data.cep,
                logradouro: response.data.logradouro,
                complemento: response.complemento,
                bairro: response.data.bairro,
                localidade: response.data.localidade,
                uf: response.data.uf
              };
              this.opcoes.error = false;

              this.items.push(this.opcoes.item);
              lodash.orderBy(this.items, ['cep'], ['asc']);
            }
          })
          .catch(erro => {
            this.msg = "Error ao fazer requisição na API VIACEP";
            this.opcoes.error = true;
          });
      }
      if (this.opcoes.cep.length != 8) {
        this.msg = "O CEP deve conter 8 Caracteres.";
        this.opcoes.error = true;
      }
    }
  },
  computed: {
    validation() {
      return this.opcoes.cep.length == 8;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
