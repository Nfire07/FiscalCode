<script>
import DataTable from '@/components/DataTable.vue';
import DynamicForm from '@/components/DynamicForm.vue';
import Fab from '@/components/Fab.vue';
import Layout from '@/components/Layout.vue';
import Modal from '@/components/Modal.vue';
import Papa from 'papaparse';

export default {
  name: 'HomeView',

  components: {
    DynamicForm,
    Layout,
    Modal,
    DataTable,
    Fab
  },

  data() {
    return {
      catastale: [],
      codiceFiscale: "",
      isCFOpen:false,
      isCTOpen:false,
      formSchema: [
        {
          type: "divider",
          name: "divider",
          label: "Dati Richiesti"
        },
        {
          type: "text",
          name: "nome",
          label: "NOME",
          placeholder: "Scrivi qui il tuo nome...",
          required: true,
          fullWidth: true,
        },
        {
          type: "text",
          name: "secondo_nome",
          label: "SECONDO NOME",
          placeholder: "Scrivi qui il tuo secondo nome...",
          required: false,
          fullWidth: true,
        },
        {
          type: "text",
          name: "cognome",
          label: "COGNOME",
          placeholder: "Scrivi qui il tuo cognome...",
          required: true,
          fullWidth: true,
        },
        {
          type: "date",
          name: "data_nascita",
          label: "DATA DI NASCITA",
          required: true,
          fullWidth: true,
        },
        {
          type: "text",
          name: "citta_nascita",
          label: "CITTÀ DI NASCITA",
          placeholder: "Scrivi qui la tua città di nascita...",
          required: true,
          fullWidth: true,
        },
        {
          type: "select",
          name: "sesso",
          label: "SESSO BIOLOGICO",
          placeholder: "Seleziona il tuo sesso biologico...",
          required: true,
          options: [
            { label: "Maschio", value: "M" },
            { label: "Femmina", value: "F" }
          ],
          fullWidth: true,
        },
      ],
    };
  },

  async mounted() {
    const catastaleFile = await fetch('/src/assets/catastale.csv');
    const text = await catastaleFile.text();
    const result = Papa.parse(text, {
      header: false,
      skipEmptyLines: true,
      dynamicTyping: false
    });
    this.catastale = result.data
      .filter(row => row.length >= 2 && row[0] && row[1])
      .map(row => ({
        codice: row[0].trim(),
        comune: row[1].trim().toUpperCase()
      }));
  },

  methods: {
    submit(data) {
      const cittaInput = data.citta_nascita.trim().toUpperCase();
      const record = this.catastale.find(row => row.comune === cittaInput);

      if (!record) {
        alert(`Comune "${data.citta_nascita}" non trovato nel database.`);
        return;
      }

      this.codiceFiscale = this.calcolaCodiceFiscale(
        data.cognome,
        data.nome,
        data.secondo_nome || '',
        data.data_nascita,
        data.sesso,
        record.codice
      );
      this.isCFOpen = true;
    },

    calcolaCodiceFiscale(cognome, nome, secondoNome, dataNascita, sesso, codiceCatastale) {
      const cf = [
        this.calcolaCognome(cognome),
        this.calcolaNome(nome, secondoNome),
        this.calcolaData(dataNascita, sesso),
        codiceCatastale.toUpperCase(),
      ].join('');

      return cf + this.calcolaControllo(cf);
    },

    calcolaCognome(cognome) {
      const s = cognome.toUpperCase().replace(/[^A-Z]/g, '');
      const consonanti = s.split('').filter(c => !'AEIOU'.includes(c));
      const vocali = s.split('').filter(c => 'AEIOU'.includes(c));
      return [...consonanti, ...vocali, 'X', 'X', 'X'].slice(0, 3).join('');
    },

    calcolaNome(nome, secondoNome) {
      const nomeCompleto = secondoNome ? nome + secondoNome : nome;
      const s = nomeCompleto.toUpperCase().replace(/[^A-Z]/g, '');
      const consonanti = s.split('').filter(c => !'AEIOU'.includes(c));
      const vocali = s.split('').filter(c => 'AEIOU'.includes(c));

      if (consonanti.length >= 4) {
        return [consonanti[0], consonanti[2], consonanti[3]].join('');
      }
      return [...consonanti, ...vocali, 'X', 'X', 'X'].slice(0, 3).join('');
    },

    calcolaData(dataNascita, sesso) {
      const d = new Date(dataNascita);
      const anno = String(d.getFullYear()).slice(2);
      const mesi = ['A', 'B', 'C', 'D', 'E', 'H', 'L', 'M', 'P', 'R', 'S', 'T'];
      const mese = mesi[d.getMonth()];
      const giorno = sesso === 'F'
        ? String(d.getDate() + 40).padStart(2, '0')
        : String(d.getDate()).padStart(2, '0');

      return anno + mese + giorno;
    },

    calcolaControllo(cf15) {
      const dispari = {
        '0': 1,  '1': 0,  '2': 5,  '3': 7,  '4': 9,
        '5': 13, '6': 15, '7': 17, '8': 19, '9': 21,
        'A': 1,  'B': 0,  'C': 5,  'D': 7,  'E': 9,
        'F': 13, 'G': 15, 'H': 17, 'I': 19, 'J': 21,
        'K': 2,  'L': 4,  'M': 18, 'N': 20, 'O': 11,
        'P': 3,  'Q': 6,  'R': 8,  'S': 12, 'T': 14,
        'U': 16, 'V': 10, 'W': 22, 'X': 25, 'Y': 24, 'Z': 23
      };
      const pari = {
        '0': 0,  '1': 1,  '2': 2,  '3': 3,  '4': 4,
        '5': 5,  '6': 6,  '7': 7,  '8': 8,  '9': 9,
        'A': 0,  'B': 1,  'C': 2,  'D': 3,  'E': 4,
        'F': 5,  'G': 6,  'H': 7,  'I': 8,  'J': 9,
        'K': 10, 'L': 11, 'M': 12, 'N': 13, 'O': 14,
        'P': 15, 'Q': 16, 'R': 17, 'S': 18, 'T': 19,
        'U': 20, 'V': 21, 'W': 22, 'X': 23, 'Y': 24, 'Z': 25
      };

      let somma = 0;
      cf15.toUpperCase().split('').forEach((c, i) => {
        somma += (i % 2 === 0) ? dispari[c] : pari[c];
      });

      return String.fromCharCode(65 + (somma % 26));
    },
  },
};
</script>

<template>
   <Layout
    type="grid"
    max-width="800px"
    padding="0 1.5rem"
    columns="1fr"
    gap="2rem"
    :center="true">

    <h1 style="text-align: center;font-weight: 800;">Calcolatore Codice Fiscale</h1>
    <DynamicForm :fields="formSchema" submit-text="Calcola il tuo Codice fiscale" @submit-form="submit"/>      
  </Layout> 

  <Modal v-model="isCFOpen" width="1000px" max-height="1000px">
      <h2> Il tuo codice fiscale è:</h2>
      <hr>
      <h1>{{ codiceFiscale }}</h1>
  </Modal>

  <Modal v-model="isCTOpen" width="1000px" max-height="1100px">
    <DataTable
      :columns="[
        { key: 'codice', label: 'Codice' },
        { key: 'comune', label: 'Comune' }
      ]"
      :rows="catastale"
      :rowsPerPageOptions="[10]"
    />
  </Modal>

  <Fab
    icon="menu"
    position="bottom-right"
    :offset="32"
    :show-labels="true"
    :actions="[
      { icon: 'info', label: 'Info', handler: () => {isCTOpen=true;console.log(catastale)} },
    ]"
  />
</template>

<style>
</style>