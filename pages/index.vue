<template lang="pug">
div
  //- Рассчитать сумму значений поля height для всех объектов
  span.text-h6 The height fields sum for all objects
  v-text-field(v-model='heightSum' label="Result" disabled = true)
  v-btn.mb-4(color="primary" dark v-bind="attrs" v-on="on") Calculate

  //- Получение объекта по id
  v-text-field(v-model="idForGettingPerson" label="Enter an id to get a person")

  v-row
    v-col
      v-text-field(v-model='itemByID.name' label="Name" disabled = true)
      v-text-field(v-model='itemByID.creation_date' label="Creation date" disabled = true)
      v-text-field(v-model='itemByID.height' label="Height" disabled = true)

    v-col
      v-text-field(v-model='itemByID.eye_color' label="Eye color" disabled = true)
      v-text-field(v-model='itemByID.hair_color' label="Hair color" disabled = true)
      v-text-field(v-model='itemByID.nationality' label="Nationality" disabled = true)

    v-col
      span.text-h8 Location
      v-text-field(v-model='itemByID.location.x' label="X" disabled = true)
      v-text-field(v-model='itemByID.location.y' label="Y" disabled = true)
      v-text-field(v-model='itemByID.location.z' label="Z" disabled = true)


    v-col
      span.text-h8 Coordinates
      v-text-field(v-model='itemByID.coordinates.x' label="X" disabled = true)
      v-text-field(v-model='itemByID.coordinates.y' label="Y" disabled = true)

  //- Выбор объектов, поля name которых содержат указанную подстроку
  v-text-field(v-model='substring' label="Enter a substring to get persons whose name contanes it")
  v-btn.mb-10(color="primary" dark v-bind="attrs" v-on="on") Filter

  //- Таблица объектов с возможностью создания новых/удаления/изменения существующих
  v-data-table.mb-5(
    :headers='headers',
    :items='persons',
  )
    template(v-slot:item.coordinates="{ item }") {{ '(' + item.coordinates.x + ', ' + item.coordinates.y + ')' }}

    template(v-slot:item.location="{ item }") {{ '(' + item.location.x + ', ' + item.location.y + ', ' + item.location.z + ')' }}

    template(v-slot:top)
      v-toolbar(flat)
        v-toolbar-title List of persons
        v-divider.mx-4(inset vertical)
        v-spacer

        v-dialog(v-model="dialog" max-width="500px")
          template(v-slot:activator="{ on, attrs }")
            v-btn.mb-2(color="primary" dark v-bind="attrs" v-on="on") Add person
          v-card
            v-card-title
              span.text-h5 {{ formTitle }}
            v-card-text
              v-container
                v-row
                  v-col(cols="12" sm="6" md="4")
                    v-text-field(v-model="editedItem.name" label="Name")
                    v-text-field(v-model="editedItem.height" label="Height")
                    v-text-field(v-model="editedItem.eye_color" label="Eye color")
                    v-text-field(v-model="editedItem.hair_color" label="Hair color")
                    v-text-field(v-model="editedItem.nationality" label="Nationality")

                  v-col(cols="12" sm="6" md="4")
                    span.text-h8 Coordinates

                    v-text-field(v-model="editedItem.coordinates.x" label="X")
                    v-text-field(v-model="editedItem.coordinates.y" label="Y")

                  v-col(cols="12" sm="6" md="4")
                    span.text-h8 Location
                    v-text-field(v-model="editedItem.location.x" label="X")
                    v-text-field(v-model="editedItem.location.y" label="Y")
                    v-text-field(v-model="editedItem.location.z" label="Z")

            v-card-actions
              v-spacer
              v-btn(color="blue darken-1" text @click="close") Cancel
              v-btn(color="blue darken-1" text @click="save")  Save

        v-dialog(v-model="dialogDelete" max-width="500px" )
          v-card
            v-card-title(class="text-h5") Are you sure you want to delete this item?
            v-card-actions
              v-spacer
              v-btn(color="blue darken-1" text @click="closeDelete") Cancel
              v-btn(color="blue darken-1" text @click="deleteItemConfirm") OK
              v-spacer

    template(v-slot:item.actions="{ item }")
      v-icon.mr-2(small @click="editItem(item)") mdi-pencil
      v-icon(small @click="deleteItem(item)") mdi-delete

    template(v-slot:no-data)
      v-btn( color="primary" @click="initialize") Reset

  //- Получение уникальных локаций
  v-data-table(
    :headers='locationHeaders',
    :items='uniqueLocations',
  )
    template(v-slot:top)
      v-toolbar(flat)
        v-toolbar-title Unique locations
        v-divider.mx-4(inset vertical)
        v-spacer
        v-btn.mb-2(color="primary" dark) Get

</template>

<script>
const xmlToJson = require("xmlToJson");
import axios from "axios";

const headers = [
  { text: "ID", value: "id" },
  { text: "Name", value: "name" },
  { text: "Coordinates", value: "coordinates" },
  { text: "Creation date", value: "creation_date" },
  { text: "Height", value: "height" },
  { text: "Eye color", value: "eye_color" },
  { text: "Hair color", value: "hair_color" },
  { text: "Nationality", value: "nationality" },
  { text: "Location", value: "location" },
  { text: "", value: "actions" }
];

const locationHeaders = [
  { text: "X", value: "x" },
  { text: "Y", value: "y" },
  { text: "Z", value: "z" }
];

export default {
  data() {
    return {
      headers,
      locationHeaders,
      persons: null,
      uniqueLocations: null,
      dialog: false,
      dialogDelete: false,
      editedIndex: -1,
      idForGettingPerson: null,
      heightSum: null,
      substring: null,
      editedItem: {
        id: "",
        name: "",
        coordinates: { x: "", y: "" },
        creation_date: "",
        height: "",
        eye_color: "",
        hair_color: "",
        nationality: "",
        location: { x: "", y: "", z: "" }
      },
      defaultItem: {
        id: "",
        name: "",
        coordinates: { x: "", y: "" },
        creation_date: "",
        height: "",
        eye_color: "",
        hair_color: "",
        nationality: "",
        location: { x: "", y: "", z: "" }
      },
      itemByID: {
        id: null,
        name: null,
        coordinates: { x: null, y: null },
        creation_date: null,
        height: null,
        eye_color: null,
        hair_color: null,
        nationality: null,
        location: { x: null, y: null, z: null }
      }
    };
  },

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    }
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    }
  },

  created() {
    this.initialize();
  },

  methods: {
    async fetchPersons() {
      data = await this.$axios.$get("api/person");

      return xmlToJson.parseString(data);
    },

    initialize() {
      this.persons = this.fetchPersons;

      // this.persons = [
      //   {
      //     id: 1,
      //     name: "Булочка",
      //     coordinates: { x: 1, y: 1 },``
      //     creation_date: "17.09.21",
      //     height: 170,
      //     eye_color: "GREEN",
      //     hair_color: "BLACK",
      //     nationality: "Magic place",
      //     location: { x: 1, y: 2, z: 3 }
      //   },
      //   {
      //     id: 2,
      //     name: "Пирожок",
      //     coordinates: { x: 1, y: 1 },
      //     creation_date: "17.09.21",
      //     height: 170,
      //     eye_color: "GREEN",
      //     hair_color: "BLACK",
      //     nationality: "Magic place",
      //     location: { x: 1, y: 2, z: 3 }
      //   },
      //   {
      //     id: 3,
      //     name: "Пончик",
      //     coordinates: { x: 1, y: 1 },
      //     creation_date: "17.09.21",
      //     height: 170,
      //     eye_color: "GREEN",
      //     hair_color: "BLACK",
      //     nationality: "Magic place",
      //     location: { x: 1, y: 2, z: 3 }
      //   },
      //   {
      //     id: 4,
      //     name: "Печенька",
      //     coordinates: { x: 1, y: 1 },
      //     creation_date: "17.09.21",
      //     height: 170,
      //     eye_color: "GREEN",
      //     hair_color: "BLACK",
      //     nationality: "Magic place",
      //     location: { x: 1, y: 2, z: 3 }
      //   },
      //   {
      //     id: 5,
      //     name: "Мандаринка",
      //     coordinates: { x: 1, y: 1 },
      //     creation_date: "17.09.21",
      //     height: 170,
      //     eye_color: "GREEN",
      //     hair_color: "BLACK",
      //     nationality: "Magic place",
      //     location: { x: 1, y: 2, z: 3 }
      //   }
      // ];

      this.uniqueLocations = [
        { x: 1, y: 2, z: 3 },
        { x: 1, y: 2, z: 3 },
        { x: 1, y: 2, z: 3 }
      ];
    },

    editItem(item) {
      this.editedIndex = this.persons.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.persons.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.persons.splice(this.editedIndex, 1);
      this.closeDelete();
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.persons[this.editedIndex], this.editedItem);
      } else {
        this.persons.push(this.editedItem);
      }
      this.close();
    }
  }
};
</script>
