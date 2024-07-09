<template>
  <div>
    <input type="file" @change="handleFileInput" accept=".xlsx, .xls" />
    <div v-if="excelData.length">
      <h3>Datos Importados:</h3>
      <table>
        <thead>
          <tr>
            <th v-for="(header, index) in headers" :key="index">{{ header }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, rowIndex) in excelData" :key="rowIndex">
            <td v-for="(cell, cellIndex) in row" :key="cellIndex">{{ cell }}</td>
          </tr>
        </tbody>
      </table>
      <button @click="syncData">Sincronizar</button>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
import { read, utils } from 'xlsx';
import axios from 'axios';

export default {
  setup() {
    const excelData = ref([]);
    const headers = ref([]);

    const handleFileInput = (event) => {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          const data = new Uint8Array(e.target.result);
          const workbook = read(data, { type: 'array' });
          const firstSheetName = workbook.SheetNames[0];
          const worksheet = workbook.Sheets[firstSheetName];
          const jsonData = utils.sheet_to_json(worksheet, { header: 1 });

          headers.value = jsonData[0];
          excelData.value = jsonData.slice(1);
        };
        reader.readAsArrayBuffer(file);
      }
    };

    const syncData = async () => {
      try {
        const response = await axios.post('https://localhost:3000/sync', {
          data: excelData.value
        });
        console.log('Datos sincronizados:', response.data);
        alert('Datos sincronizados con éxito.');
      } catch (error) {
        console.error('Error al sincronizar los datos:', error);
        alert('Hubo un error al sincronizar los datos.');
      }
    };

    return {
      excelData,
      headers,
      handleFileInput,
      syncData
    };
  }
};
</script>

<style>
table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  background-color: #f2f2f2;
}

button {
  margin-top: 10px;
  padding: 10px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}
</style>
