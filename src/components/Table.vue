<template>
  <v-container class="fill-height" fluid>
    <v-row align="center" justify="center">
      <v-col cols="12" sm="8">
        <v-card class="elevation-12">
          <v-btn class="ma-2" tile color="indigo" dark @click="createData">
            Добавить
          </v-btn>
          <v-simple-table dense fixed-header height="450">
            <template v-slot:default>
              <thead>
                <tr>
                  <th class="text-left">Название</th>
                  <th class="text-left">Дата</th>
                  <th class="text-left">Количество</th>
                  <th class="text-left">Наличие</th>
                  <th class="text-left"></th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in db" :key="item.id">
                  <td>
                    <v-text-field
                      v-if="item.isEdit"
                      ref="name"
                      type="text"
                      placeholder="Название"
                      :value="currentFields.name"
                    ></v-text-field>
                    <span v-else>
                      {{ item.name }}
                    </span>
                  </td>
                  <td>
                    <v-menu
                      v-if="item.isEdit"
                      :close-on-content-click="false"
                      transition="scale-transition"
                      offset-y
                      max-width="290px"
                      min-width="290px"
                    >
                      <template v-slot:activator="{ on }">
                        <v-text-field
                          :value="currentFields.date"
                          ref="date"
                          label="Дата"
                          prepend-icon="mdi-calendar"
                          readonly
                          v-on="on"
                        ></v-text-field>
                      </template>
                      <v-date-picker
                        v-model="currentFields.date"
                        no-title
                      ></v-date-picker>
                    </v-menu>
                    <span v-else>
                      <v-icon>mdi-calendar</v-icon>
                      {{ item.date }}
                    </span>
                  </td>
                  <td>
                    <v-text-field
                      v-if="item.isEdit"
                      type="number"
                      ref="number"
                      :value="currentFields.number"
                    />
                    <span v-else>{{ item.number }}</span>
                  </td>
                  <td>
                    <v-checkbox
                            v-if="item.isEdit"
                            ref="check"
                            v-model="currentFields.check"
                            :ripple="false"
                    ></v-checkbox>
                    <v-checkbox
                            v-else
                      v-model="item.check"
                      :disabled="!item.isEdit"
                      :ripple="false"
                    ></v-checkbox>
                  </td>
                  <td>
                    <v-btn
                      v-if="item.isEdit"
                      icon
                      small
                      @click="
                        changeData(item.id)
                      "
                    >
                      <v-icon dark color="blue darken-2">
                        mdi-content-save
                      </v-icon>
                    </v-btn>
                    <v-btn v-else icon small @click="editMode(item.id)">
                      <v-icon dark color="blue darken-2">
                        mdi-pencil
                      </v-icon>
                    </v-btn>
                    <v-btn icon small @click="deleteData(item.id)">
                      <v-icon dark color="red darken-2">
                        mdi-delete
                      </v-icon>
                    </v-btn>
                  </td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";

const instance = axios.create({
  baseURL: "http://localhost:3000/api/tests"
});

const initialFields = {
  name: "",
  date: new Date().toISOString(),
  number: 0,
  check: false
};

export default {
  data() {
    return {
      db: [],
      currentFields: initialFields
    };
  },
  mounted() {
    instance.get("/").then(response => {
      return (this.db = response.data.map(i => {
        i.date = i.date.substr(0, 10);
        return { isEdit: false, ...i };
      }));
    });
  },
  methods: {
    createData() {
      let data = initialFields;
      instance
        .post("/", JSON.stringify(data), {
          headers: { "content-type": "application/json" }
        })
        .then(response => {
          response.data.date = response.data.date.substr(0, 10);
          this.currentFields = response.data;
          this.db = this.db.map(d => {
            d.isEdit = false;
            return d;
          });
          return this.db.push({ isEdit: true, ...response.data });
        });
    },
    changeData(id) {
      let data = {
        name: this.$refs.name[0].$refs.input._value,
        date: new Date(this.$refs.date[0].$refs.input._value).toISOString(),
        number: this.$refs.number[0].$refs.input._value,
        check: this.$refs.check[0].$refs.input.checked
      }
      instance
        .patch(`/${id}`, data, {
          headers: { "content-type": "application/json" }
        })
        .then(response => {
          return (this.db = this.db.map(d => {
            response.data.date = response.data.date.substr(0, 10);
            this.currentFields = initialFields;
            if (d.id === id) return { isEdit: false, ...response.data };
            return d;
          }));
        });
    },
    deleteData(id) {
      instance.delete(`/${id}`).then(response => {
        if (response.data.count > 0) this.db = this.db.filter(d => d.id !== id);
        return response.data.count;
      });
    },
    editMode(id) {
      this.db = this.db.map(i => {
        if (i.id === id) {
          i.isEdit = true;
          this.currentFields = { ...i };
        } else i.isEdit = false;
        return i;
      });
    }
  }
};
</script>
