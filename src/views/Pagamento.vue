<template>
  <v-container fluid>
    <div>
      <v-snackbar
        type="info"
        v-model="compraEfetuada"
        close-text="Close Alert"
        color="info"
        :top="y === 'top'"
      >PARABÉNS, SUA COMPRA FOI FINALIZADA! EM BREVE CHEGARÁ NO LOCAL DE ENTREGA. OBRIGADO POR COMPRAR COM O SMART MARKET!</v-snackbar>
    </div>
    <v-row align="center">
      <v-col class="mx-auto" cols="12" sm="6">
        <v-select :items="items" filled label="Forma de Pagamento" v-model="formadepagamento"></v-select>
        <v-list>
          <v-list-item-title class="title">
            Valor total da compra :
            <v-list-item-title>R$: {{valortotaldaCompra}},00</v-list-item-title>
          </v-list-item-title>

          <v-list-item-title class="title" v-if="formadepagamento == 'Cartão de Crédito'">
            Valor total das Parcelas :
            <v-list-item-title>R$: {{valordasParcelas}}</v-list-item-title>
          </v-list-item-title>

          <v-list-item-title class="title" v-if="formadepagamento == 'Cartão de Crédito'">
            Quantidade de Parcelas :
            <v-list-item-title>{{qtdParcelas}}x</v-list-item-title>
          </v-list-item-title>

          <template>
            <div class="text-center">
              <v-btn
                :disabled="dialog1"
                :loading="dialog1"
                class="white--text"
                color="purple darken-2"
                @click="selecPagamento"
              >Processar Pagamento</v-btn>
              <v-dialog v-model="dialog1" hide-overlay persistent width="300" v-if="selecPagamento">
                <v-card color="primary" dark>
                  <v-card-text>
                    Por favor, aguarde alguns segundos e te redicionaremos à pagina inicial do aplicativo...
                    <v-progress-linear indeterminate color="white" class="mb-0"></v-progress-linear>
                  </v-card-text>
                </v-card>
              </v-dialog>
            </div>
          </template>
        </v-list>
        <template>
          <v-row justify="center">
            <v-dialog
              v-if="formadepagamento == 'Cartão de Crédito'"
              v-model="dialog"
              scrollable
              max-width="350px"
            >
              <template v-slot:activator="{ on }">
                <v-btn
                  color="primary"
                  dark
                  v-on="on"
                >Selecione o Número de Parcelas (Até 3x sem juros)</v-btn>
              </template>
              <v-card>
                <v-card-title>Selecione o Número de Parcelas</v-card-title>
                <v-divider></v-divider>
                <v-card-text style="height: 300px;">
                  <v-radio-group v-model="dialogm1" column>
                    <v-radio label="1x - " value="1"></v-radio>
                    <v-radio label="2x - " value="2"></v-radio>
                    <v-radio label="3x - " value="3"></v-radio>
                    <v-radio label="4x - " value="4"></v-radio>
                    <v-radio label="5x - " value="5"></v-radio>
                  </v-radio-group>
                </v-card-text>
                <v-divider></v-divider>
                <v-card-actions>
                  <v-btn color="blue darken-1" text @click="fechardialog">Fechar</v-btn>
                  <v-btn color="blue darken-1" text @click="calcularParcelas()">Salvar</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-row>
        </template>

        <template>
          <v-alert type="warning" v-if="alertaPag">Selecione uma forma de Pagamento!</v-alert>
        </template>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import HttpRequestUtil from "@/util/HttpRequestUtil";
export default {
  data: () => ({
    dialogm1: 0,
    dialog: false,
    dialog1: false,
    alertaPag: false,
    compraEfetuada: false,

    compra: [],
    carrinho: [],

    items: ["Boleto", "Cartão de Débito", "Cartão de Crédito", "Paypal"],

    qtdParcelas: 0,

    valortotaldaCompra: 0,
    valordasParcelas: 0,
    formadepagamento: null,
    y: "center"
  }),

  watch: {
    dialog1(val) {
      if (!val) return;

      setTimeout(
        () => (
          (this.dialog1 = false),
          (this.compraEfetuada = true),
          this.$router.push("/")
        ),
        8000
      );
    }
  },

  methods: {
    fechardialog() {
      this.formadepagamento = null;
      this.dialog = false;
    },

    calcularParcelas() {
      if (this.formadepagamento == "Cartão de Crédito" && this.dialogm1 > 0) {
        if (this.dialogm1 <= 3) {
          this.valordasParcelas = this.valortotaldaCompra / this.dialogm1;
        } else {
          this.valordasParcelas =
            this.valortotaldaCompra / this.dialogm1 +
            (5 / 100) * (this.valortotaldaCompra / this.dialogm1);
        }

        this.qtdParcelas = this.dialogm1;
        this.dialog = false;
      }
    },

    atualizarEstoque(compraNova) {
      HttpRequestUtil.atualizarQuantidade(compraNova).then(response => {alert("quantidade atualizada")});
    },

    selecPagamento() {
      let produto = JSON.parse(localStorage.getItem("carrinho"));
      let cliente = JSON.parse(localStorage.getItem("clienteLogado"));
      if (this.formadepagamento == null) {
        this.alertaPag = true;
      } else {
        this.alertaPag = false;

        let compraNova = {};

        compraNova.cliente = cliente;
        compraNova.produtos = produto;
        compraNova.data = "12/12/2020";
        compraNova.valorTotal = this.valortotaldaCompra;
        compraNova.pagamento = this.formadepagamento;

        this.compraEfetuada = true;

        HttpRequestUtil.salvarCompras(compraNova).then(compraRetornada => {
          localStorage.setItem("carrinho", JSON.stringify(this.carrinho));
          this.atualizarEstoque();
          this.dialog1 = true;
        });
      }
    }
  },

  mounted() {
    if (localStorage.getItem("carrinho") == null) {
      localStorage.setItem("carrinho", JSON.stringify(this.compra));
    } else {
      let compra = JSON.parse(localStorage.getItem("carrinho"));
      this.compra = compra;
    }
    for (let i = 0; i < this.compra.length; i++) {
      this.valortotaldaCompra += parseFloat(this.compra[i].valor);
    }
  }
};
</script>