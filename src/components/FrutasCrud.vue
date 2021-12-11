<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <h1>Lista de Frutas</h1>
            <v-data-table
              :headers="headers"
              :items="frutas"
              sort-by="calories"
              class="elevation-1"
            >
            <template v-slot:top> 
            <v-toolbar flat>
              <v-spacer></v-spacer>
              <v-dialog v-model="dialog" max-width="700px">
                <template v-slot:activator="{ on, attrs }">
                  <v-btn
                    color="primary"
                    dark
                    class="mb-2"
                    v-bind="attrs"
                    v-on="on"
                  >
                    Novo Registro
                  </v-btn>
                </template>
                <v-card>
                  <v-card-title>
                    <span class="headline">FrutaCrud</span>
                  </v-card-title>
                 <v-card-text>
                      <v-container>
                        <v-row>
                          <v-col cols="12" sm="6" md="4">
                            <v-text-field
                              v-model="editedItem.id"
                              label="Id"
                              type="number"
                                                         
                            ></v-text-field>
                          </v-col>
                          <v-col cols="12" sm="6" md="8">
                            <v-text-field
                              v-model="editedItem.nome"
                              label="Nome"
                               
                            ></v-text-field>
                          </v-col>
                          <v-col cols="12" sm="6" md="4">
                            <v-text-field
                              v-model="editedItem.calorias"
                              label="Calorias"
                              suffix="kcal"
                            ></v-text-field>
                          </v-col>                             
                        </v-row>
                      </v-container>
                    </v-card-text>

                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="close">
                      Cancel
                    </v-btn>
                    <v-btn color="blue darken-1" text @click="save">
                      Save
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>
            </v-toolbar>
          </template>
          <template v-slot:[`item.calorias`]="{ item }">
          {{item.calorias}}kcal
          </template>
           <template v-slot:[`item.actions`]="{ item }">
              <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
              <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
            </template>
            <template v-slot:no-data>
              <v-btn color="primary" @click="initialize"> Reset 
              </v-btn>
            </template>
          </v-data-table>
        </v-col>
      </v-row>
    </v-container>
</template>

<script>
import axios from "axios";
export default {
  name: "FrutasCrud",
  data: () => ({
    dialog: false,
    headers: [
      { text: "Id", value: "id" },
      { text: "Nome", value: "nome" },
      { text: "Calorias", value: "calorias" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    frutas: [],
    editedIndex: -1,
    editedItem: {
      id: 0,
      nome: "",
      calorias: 0,
    },
    defaultItem: {
      id: 0,
      nome: "",
      calorias: 100,
    },
  }),
  methods: {
    inicializa() {
      axios("http://localhost:3000/frutas")
        .then((response) => {
          this.frutas = response.data;
        })
        .catch((error) => console.log(error));
    },
    close() {
      this.dialog = false;
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      }, 300);
    },
    save() {
      if (this.editedIndex > -1) {
        //alteracao
        axios
          .put(
            "http://localhost:3000/frutas/" + this.editedItem.id,
            this.editedItem
          )
          .then((response) => {
            console.log(response);
            Object.assign(this.frutas[this.editedIndex], this.editedItem);
            this.close();
          })
          .catch((error) => console.log(error));
      } else {
        //Inclusao
        axios
          .post("http://localhost:3000/frutas", this.editedItem)
          .then((response) => {
            console.log(response);
            this.frutas.push(this.editedItem);
            this.close();
          })
          .catch((error) => console.log(error));
      }
    },
    editItem(item) {
      this.editedIndex = this.frutas.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },
    deleteItem(item) {
      const index = this.frutas.indexOf(item);
      confirm("Deseja apagar este item?") &&
        axios
          .delete("http://localhost:3000/frutas/" + item.id)
          .then((response) => {
            console.log(response.data);
            this.frutas.splice(index, 1);
          })
          .catch((error) => console.log(error));
    },
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Novo Registro" : "Editar Item";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },
  created() {
    this.inicializa();
  },
};
</script>

<style scoped>
</style>