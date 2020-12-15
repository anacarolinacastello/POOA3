<template>
  <v-data-table
    :headers="headers"
    :items="frutas"
    sort-by="calories"
    class="elevation-1"
  >
    
    <template v-slot:[`item.actions`]="{ item }">
      <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
      <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="initialize"> Reset </v-btn>
    </template>
  </v-data-table>
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
      console.log(item.id)
    },
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Novo Item" : "Editar Item";
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