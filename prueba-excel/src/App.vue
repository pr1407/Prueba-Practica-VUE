<template>
  <div class="table-container">
    <h1>Tabla Excel</h1>
    <input id="archivoExcel" type="file" @change="subirExcel" accept=".xlsx" />
    <div class="table-scroll">
      <v-data-table
        :headers="headers[text]"
        :items="items"
        :items-per-page="perPage"
        :hide-default-footer="items.length <= perPage"
        class="elevation-1"
      >
      </v-data-table>
    </div>
    <div class="boton-Enviar">
      <v-btn @click="sendData" v-if="items.length">Enviar Datos</v-btn>
    </div>
    <div v-if="loading">
      <v-progress-linear
      color="yellow-darken-2"
      indeterminate
    ></v-progress-linear>
    </div>
  </div>
</template>

<script>
import readXlsxFile from "read-excel-file";
import axios from "axios";

export default {
  data() {
    return {
      perPage: 20,
      items: [],
      headers: [],
      loading: false,
      error: "",
    };
  },
  methods: {
    async subirExcel(event) {
      const input = event.target;
      if (input.files.length === 0) return;

      this.loading = true;
      this.error = "";
      this.items = [];
      this.headers = [];
      this.ignore = ["Ignorar"];

      console.log("entro");
      try {
        const rows = await readXlsxFile(input.files[0], { sheet: 1 });

        if (rows.length === 0) throw new Error("El archivo Excel está vacío.");

        // Captura y establece los encabezados
        this.headers = rows[0].map((header) => ({
          value: header,
        }));

        // Procesa las filas del Excel para convertirlas en objetos
        this.items = rows.slice(1).map((row) => {
          const item = {};
          row.forEach((cell, index) => {
            if (!this.ignore.includes(this.headers[index].value)) {
              if (cell instanceof Date) {
                const formattedDate =
                  cell.getDate().toString().padStart(2, "0") +
                  "/" +
                  (cell.getMonth() + 1).toString().padStart(2, "0") +
                  "/" +
                  cell.getFullYear().toString();
                item[this.headers[index].value] = formattedDate;
              }else{
                item[this.headers[index].value] = cell;
              }
            }
          });
          return item;
        });

      } catch (error) {
        this.error = "Error al procesar el archivo: " + error.message;
      } finally {
        this.loading = false;
      }
    },
    async sendData() {
      try {
        const cleanData = JSON.parse(JSON.stringify(this.items));
        console.log("Datos limpios:", cleanData);
        const response = await axios.post(
          "https://pruebaSubida/Excel",
          cleanData
        );
        if (response.status === 200) {
          alert("Datos enviados exitosamente");
        } else {
          throw new Error(
            "Error en la respuesta del servidor: " + response.statusText
          );
        }
      } catch (error) {
        console.error("Error completo:", error); // Mejora el manejo de errores
        this.error = "Error al enviar datos: " + error.message;
        alert(this.error);
      }
    },
  },
};
</script>

<style scoped>
/* Estilos opcionales para la tabla */
table {
  width: 100%;
  border-collapse: collapse;
}

.table-container {
  max-width: 100%;
  overflow-x: auto;
}

.table-scroll {
  overflow-x: auto;
  max-width: 100%;
}

th,
td {
  border: 1px solid #ccc;
  padding: 8px;
  text-align: left;
}

input {
  width: 100%;
  padding: 10px;
  margin: 10px 0;
  box-sizing: border-box;
  border: 1px solid #ccc;
  border-radius: 4px;
}


input:focus {
  border-color: #4CAF50;
  outline: none;
}


.boton-Enviar {
  margin-top: 20px;
  text-align: center;
  margin-bottom: 20px;
}
</style>
