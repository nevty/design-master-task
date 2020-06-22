<template>
  <v-container class="fill-height" fluid>
    <v-row align="center" justify="center">
      <v-col cols="12" sm="8">
        <v-card class="elevation-12">
          <v-btn class="ma-2" tile color="indigo" dark @click="postData">
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
                  <td v-if="item.isEdit">
                    <v-text-field
                      type="text"
                      placeholder="Название"
                      v-model="item.name"
                      value="{ item.name }"
                    ></v-text-field>
                  </td>
                  <td v-else>{{ item.name }}</td>
                  <td v-if="item.isEdit">
                    <v-menu
                      :close-on-content-click="false"
                      transition="scale-transition"
                      offset-y
                      max-width="290px"
                      min-width="290px"
                    >
                      <template v-slot:activator="{ on }">
                        <v-text-field
                          v-model="item.date"
                          label="Дата"
                          prepend-icon="mdi-calendar"
                          readonly
                          v-on="on"
                        ></v-text-field>
                      </template>
                      <v-date-picker
                        v-model="item.date"
                        no-title
                      ></v-date-picker>
                    </v-menu>
                  </td>
                  <td v-else><v-icon>mdi-calendar</v-icon> {{ item.date }}</td>
                  <td v-if="item.isEdit">
                    <v-text-field
                      type="number"
                      v-model="item.number"
                      value="{ item.number }"
                    />
                  </td>
                  <td v-else>{{ item.number }}</td>
                  <td>
                    <v-checkbox
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
                        changeData(item.id, {
                          name: item.name,
                          date: item.date,
                          number: item.number,
                          check: item.check
                        })
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

export default {
  data() {
    return {
      db: []
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
    postData() {
      let data = {
        name: "",
        date: new Date().toISOString(),
        number: 0,
        check: false
      };
      instance
        .post("/", JSON.stringify(data), {
          headers: { "content-type": "application/json" }
        })
        .then(response => {
          response.data.date = response.data.date.substr(0, 10);
          return this.db.push({ isEdit: true, ...response.data });
        });
    },
    changeData(id, data) {
      instance
        .patch(`/${id}`, data, {
          headers: { "content-type": "application/json" }
        })
        .then(response => {
          return (this.db = this.db.map(d => {
            response.data.date = response.data.date.substr(0, 10);
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
        if (i.id === id) i.isEdit = true;
        return i;
      });
    }
  }
};
</script>
