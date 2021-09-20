<template lang="pug">
div
  //- Рассчитать сумму значений поля height для всех объектов
  v-card.mb-5
    v-card-title The height fields sum for all objects
      v-spacer
      v-btn.mb-4(color="primary", dark, @click="getHeightSum") Calculate
    v-card-text
      v-text-field(v-model="heightSum", label="Result", disabled)

  //- Получение объекта по id
  v-card.mb-5
    v-card-title Get person by id
      v-spacer
      v-btn(color="primary", @click="getPersonById") Get

    v-card-text
      v-text-field(
        v-model="idForGettingPerson",
        label="Enter an id to get a person"
      )
      v-alert(v-model="getPersonAlertOn", type="error") {{ this.error }}

      v-row
        v-col
          v-text-field(v-model="itemByID.name", label="Name", disabled)
          v-text-field(
            v-model="itemByID.creationDate",
            label="Creation date",
            disabled
          )
          v-text-field(v-model="itemByID.height", label="Height", disabled)

        v-col
          v-text-field(
            v-model="itemByID.eyeColor",
            label="Eye color",
            disabled
          )
          v-text-field(
            v-model="itemByID.hairColor",
            label="Hair color",
            disabled
          )
          v-text-field(
            v-model="itemByID.nationality",
            label="Nationality",
            disabled
          )

        v-col
          span.text-h8 Location
          v-text-field(v-model="itemByID.location.x", label="X", disabled)
          v-text-field(v-model="itemByID.location.y", label="Y", disabled)
          v-text-field(v-model="itemByID.location.z", label="Z", disabled)
        v-col
          span.text-h8 Coordinates
          v-text-field(v-model="itemByID.coordinates.x", label="X", disabled)
          v-text-field(v-model="itemByID.coordinates.y", label="Y", disabled)

  //- Выбор объектов, поля name которых содержат указанную подстроку
  v-card.mb-5
    v-card-title Filter persons by name substring
      v-spacer
      v-btn.mb-10(color="primary", dark, @click="getPersonsBySubstring") Filter
    v-card-text
      v-text-field(
        v-model="substring",
        label="Enter a substring to get persons whose name contanes it"
      )

  //- Таблица объектов с возможностью создания новых/удаления/изменения существующих
  v-alert(v-model="tableAlertOn", type="error") {{ this.error }}
  v-data-table.mb-5(
    :headers="headers",
    :items="persons",
    :items-per-page="itemsPerPage",
    hide-default-footer
  )
    template(v-slot:item.coordinates="{ item }") {{ '(' + item.coordinates.x + ', ' + item.coordinates.y + ')' }}

    template(v-slot:item.location="{ item }") {{ '(' + item.location.x + ', ' + item.location.y + ', ' + item.location.z + ')' }}

    template(v-slot:top)
      v-toolbar(flat)
        v-toolbar-title List of persons
        v-divider.mx-4(inset, vertical)
        v-spacer
        v-row(align="center")
          v-col.d-flex(cols="12", sm="5")
          v-col.d-flex(cols="12", sm="6")
            v-select.mt-8(
              :items="sortItems",
              v-model="currentSort",
              label="Sort",
              @change="initialize",
              dark
            )

        v-dialog(v-model="dialog", max-width="500px")
          template(v-slot:activator="{ on, attrs }")
            v-btn.mb-2(color="primary", dark, v-bind="attrs", v-on="on") Add person
          v-card
            v-card-title
              span.text-h5 {{ formTitle }}
            v-card-text
              v-container
                v-row
                  v-col(cols="12", sm="6", md="4")
                    v-text-field(v-model="editedItem.name", label="Name")
                    v-text-field(v-model="editedItem.height", label="Height")
                    v-select(
                      v-model="editedItem.eyeColor",
                      :items="availableColors",
                      label="Eye color"
                    )
                    v-select(
                      v-model="editedItem.hairColor",
                      :items="availableColors",
                      label="Hair color"
                    )
                    v-select(
                      v-model="editedItem.nationality",
                      :items="availableCountries",
                      label="Nationality"
                    )

                  v-col(cols="12", sm="6", md="4")
                    span.text-h8 Coordinates

                    v-text-field(v-model="editedItem.coordinates.x", label="X")
                    v-text-field(v-model="editedItem.coordinates.y", label="Y")

                  v-col(cols="12", sm="6", md="4")
                    span.text-h8 Location
                    v-text-field(v-model="editedItem.location.x", label="X")
                    v-text-field(v-model="editedItem.location.y", label="Y")
                    v-text-field(v-model="editedItem.location.z", label="Z")

            v-card-actions
              v-spacer
              v-btn(color="blue darken-1", text, @click="close") Cancel
              v-btn(color="blue darken-1", text, @click="save") Save

        v-dialog(v-model="dialogDelete", max-width="500px")
          v-card
            v-card-title.text-h5 Are you sure you want to delete this item?
            v-card-actions
              v-spacer
              v-btn(color="blue darken-1", text, @click="closeDelete") Cancel
              v-btn(color="blue darken-1", text, @click="deleteItemConfirm") OK
              v-spacer

    template(v-slot:item.actions="{ item }")
      v-icon.mr-2(small, @click="editItem(item)") mdi-pencil
      v-icon(small, @click="deleteItem(item)") mdi-delete

    template(v-slot:no-data)
      v-btn(color="primary", @click="reset") Reset
  .text-center.pt-2
    v-pagination(
      v-model="page",
      :length="pageCount",
      :total-visible="7",
      @input="initialize"
    )
    v-text-field(
      :value="itemsPerPage",
      label="Items per page",
      type="number",
      min="0",
      max="15",
      @input="itemsPerPage = parseInt($event, 10)",
      @change="initialize"
    )
  //- Выбор объектов, поля name которых содержат указанную подстроку
  v-card.mb-5
    v-card-title Advanced filter
      v-spacer

    v-card-text
      v-container
        v-row
          v-col(cols="12", sm="6", md="4")
            v-text-field(
              @change="initialize",
              v-model="filter.name",
              label="Name"
            )
            v-text-field(
              @change="initialize",
              v-model="filter.height",
              label="Height"
            )
            v-select(
              @change="initialize",
              v-model="filter.eyeColor",
              :items="availableColors",
              label="Eye color"
            )
            v-select(
              @change="initialize",
              v-model="filter.hairColor",
              :items="availableColors",
              label="Hair color"
            )
            v-select(
              @change="initialize",
              v-model="filter.nationality",
              :items="availableCountries",
              label="Nationality"
            )
            v-text-field(
              @change="initialize",
              v-model="filter.creationDate",
              label="Creation Date"
            )

          v-col(cols="12", sm="6", md="4")
            span.text-h8 Coordinates

            v-text-field(
              @change="initialize",
              v-model="filter.coordinates.x",
              label="X"
            )
            v-text-field(
              @change="initialize",
              v-model="filter.coordinates.y",
              label="Y"
            )

          v-col(cols="12", sm="6", md="4")
            span.text-h8 Location
            v-text-field(
              @change="initialize",
              v-model="filter.location.x",
              label="X"
            )
            v-text-field(
              @change="initialize",
              v-model="filter.location.y",
              label="Y"
            )
            v-text-field(
              @change="initialize",
              v-model="filter.location.z",
              label="Z"
            )

  //- Получение уникальных локаций
  v-data-table(:headers="locationHeaders", :items="uniqueLocations")
    template(v-slot:top)
      v-toolbar(flat)
        v-toolbar-title Unique locations
        v-divider.mx-4(inset, vertical)
        v-spacer
        v-btn.mb-2(color="primary", dark, @click="getUniqueLocations") Get
</template>

<script>
var { parseString, Builder } = require("xml2js");
var builder = new Builder();

const headers = [
  { text: "ID", value: "id" },
  { text: "Name", value: "name" },
  { text: "Coordinates", value: "coordinates" },
  { text: "Creation date", value: "creationDate" },
  { text: "Height", value: "height" },
  { text: "Eye color", value: "eyeColor" },
  { text: "Hair color", value: "hairColor" },
  { text: "Nationality", value: "nationality" },
  { text: "Location", value: "location" },
  { text: "", value: "actions" },
];

const locationHeaders = [
  { text: "X", value: "x" },
  { text: "Y", value: "y" },
  { text: "Z", value: "z" },
];

export default {
  data() {
    return {
      error: null,
      headers,
      locationHeaders,
      persons: [],
      uniqueLocations: [],
      dialog: false,
      dialogDelete: false,
      editedIndex: -1,
      idForGettingPerson: null,
      heightSum: null,
      substring: null,
      page: 1,
      pageCount: 1,
      itemsPerPage: 10,
      currentSort: "id",
      itemsCount: 0,
      getPersonAlertOn: false,
      tableAlertOn: false,
      availableColors: [
        "GREEN", "WHITE", "BLACK", "YELLOW", "ORANGE"
      ],
      availableCountries: [
        "INDIA", "VATICAN", "NORTH_KOREA", "JAPAN"
      ],
      sortItems: [
        "id",
        "name",
        "height",
        "eyeColor",
        "hairColor",
        "nationality",
        "creationDate",
        "coordinates_x",
        "coordinates_y",
        "location_x",
        "location_y",
        "location_z",
      ],
      filter: {
        name: "",
        coordinates: { x: "", y: "" },
        creationDate: "",
        height: "",
        eyeColor: "",
        hairColor: "",
        nationality: "",
        location: { x: "", y: "", z: "" },
      },
      editedItem: {
        id: "",
        name: "",
        coordinates: { x: "", y: "" },
        creationDate: "",
        height: "",
        eyeColor: "",
        hairColor: "",
        nationality: "",
        location: { x: "", y: "", z: "" },
      },
      defaultItem: {
        id: "",
        name: "",
        coordinates: { x: "", y: "" },
        creationDate: "",
        height: "",
        eyeColor: "",
        hairColor: "",
        nationality: "",
        location: { x: "", y: "", z: "" },
      },
      itemByID: {
        id: null,
        name: null,
        coordinates: { x: null, y: null },
        creationDate: null,
        height: null,
        eyeColor: null,
        hairColor: null,
        nationality: null,
        location: { x: null, y: null, z: null },
      },
    };
  },

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
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

  async created() {
    await this.initialize();
  },

  methods: {
    async initialize() {
      this.recountPages();

      let parameters = [
        { name: "page_id", value: this.page },
        { name: "page_size", value: this.itemsPerPage },
        { name: "sort", value: this.currentSort },
        { name: "name", value: this.filter.name },
        { name: "eye_color", value: this.filter.eyeColor.toUpperCase() },
        { name: "hair_color", value: this.filter.hairColor.toUpperCase() },
        { name: "nationality", value: this.filter.nationality.toUpperCase() },
        { name: "creation_date", value: this.filter.creationDate },
        { name: "height", value: this.filter.height },
        { name: "location_x", value: this.filter.location.x },
        { name: "location_y", value: this.filter.location.y },
        { name: "location_z", value: this.filter.location.z },
        { name: "coordinates_x", value: this.filter.coordinates.x },
        { name: "coordinates_y", value: this.filter.coordinates.y },
      ];

      const response = await this.$axios.$get(
        "/api/person?" +
          parameters
            .filter((param) => param.value != "")
            .map((param) => param.name + "=" + param.value)
            .join("&")
      );

      let itemsPerPage = this.itemsPerPage;
      let page = this.page;

      let persons = [];
      let count = 0;
      parseString(response, { explicitArray: false }, function (err, result) {
        count = result.persons.$.count;
        let recv_count = Math.min(
          Math.max(count - (page - 1) * itemsPerPage, 0),
          itemsPerPage
        );
        if (recv_count != 0) {
          if (recv_count > 1) {
            persons = result.persons.person; // returned more than one value
          } else {
            persons.push(result.persons.person); // returned one value
          }
        }
      });


      this.itemsCount = count;
      this.persons = persons;
      this.recountPages();
    },

    async getHeightSum() {
      const response = await this.$axios.$get("/api/calc_height");

      let heightSum;
      parseString(response, { explicitArray: false }, function (err, result) {
        heightSum = result.server_response.body._;
      });

      this.heightSum = heightSum;
    },

    async getPersonById() {
      this.getPersonAlertOn = false;
      try {
        const response = await this.$axios.$get(
          "/api/person?id=" + this.idForGettingPerson
        );

        let person;
        parseString(response, { explicitArray: false }, function (err, result) {
          person = result.person;
        });

        this.itemByID = person;
      } catch (error) {
        this.error = error.response.data;
        this.getPersonAlertOn = true;
      }
    },

    async getPersonsBySubstring() {
      const response = await this.$axios.$get(
        "/api/search_by_name?name=" + this.substring
      );

      let persons = [];
      parseString(response, { explicitArray: false }, function (err, result) {
        if (result.persons != "") {
          // returned empty list
          if (result.persons.item.constructor === Array) {
            persons = result.persons.item; // returned more than one value
          } else {
            persons.push(result.persons.item); // returned one value
          }
        }
      });

      this.persons = persons;
    },

    async getUniqueLocations() {
      const response = await this.$axios.$get("/api/uniq_locations");

      let locations = [];
      parseString(response, { explicitArray: false }, function (err, result) {
        if (result.locations != "") {
          // returned empty list
          if (result.locations.item.constructor === Array) {
            locations = result.locations.item; // returned more than one value
          } else {
            locations.push(result.locations.item); // returned one value
          }
        }
      });

      this.uniqueLocations = locations;
    },

    editItem(item) {
      this.editedIndex = this.persons.indexOf(item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.persons.indexOf(item);
      this.dialogDelete = true;
    },

    async deleteItemConfirm() {
      await this.deletePerson();
      await this.initialize();
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

    async createPerson() {
      try {
        await this.$axios.$post(
          "/api/person",
          builder.buildObject({
            person: {
              name: this.editedItem.name,
              height: this.editedItem.height,
              eyeColor: this.editedItem.eyeColor.toUpperCase(),
              hairColor: this.editedItem.hairColor.toUpperCase(),
              nationality: this.editedItem.nationality.toUpperCase(),
              coordinates: this.editedItem.coordinates,
              location: this.editedItem.location,
            },
          })
        );
      } catch (error) {
        let json_error = "";
        parseString(error.response.data, {explicitArray: false}, function (err, result) {
          json_error = result;
        });
        this.error = json_error;
        this.tableAlertOn = true;
      }
    },

    async updatePerson() {
      try {
        await this.$axios.$put(
          "/api/person?id=" + this.persons[this.editedIndex].id,
          builder.buildObject({
            person: {
              name: this.editedItem.name,
              height: this.editedItem.height,
              eyeColor: this.editedItem.eyeColor.toUpperCase(),
              hairColor: this.editedItem.hairColor.toUpperCase(),
              nationality: this.editedItem.nationality.toUpperCase(),
              coordinates: this.editedItem.coordinates,
              location: this.editedItem.location,
            },
          })
        );
      } catch (error) {
        this.error = error.response.data;
        this.tableAlertOn = true;
      }
    },

    async deletePerson() {
      await this.$axios.$delete(
        "/api/person?id=" + this.persons[this.editedIndex].id
      );
    },

    async save() {
      this.tableAlertOn = false;
      if (this.editedIndex > -1) {
        Object.assign(this.persons[this.editedIndex], this.editedItem);
        await this.updatePerson();
      } else {
        await this.createPerson();
      }
      await this.initialize();
      this.close();
    },

    async reset() {
      this.page = 1;
      this.pageCount = 1;
      this.itemsPerPage = 10;
      this.currentSort = "id";
      this.filter = {
        name: "",
        coordinates: { x: "", y: "" },
        creationDate: "",
        height: "",
        eyeColor: "",
        hairColor: "",
        nationality: "",
        location: { x: "", y: "", z: "" },
      };

      await this.initialize();
    },

    recountPages() {
      this.pageCount = Math.max(Math.ceil(this.itemsCount / this.itemsPerPage), 1);
      this.page = Math.min(this.page, this.pageCount);
    },
  },
};
</script>
