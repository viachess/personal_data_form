<template>
  <v-col>
    <h3 class="mb-4">Паспортные данные</h3>
    <v-row>
      <v-col>
        <v-autocomplete
          name="citizenship"
          solo
          :items="filteredNationalities"
          v-on:update:search-input="throttledDisplayFiltered"
          no-filter
          v-on:input="passportFieldsController"
          label="Гражданство"
          :rules="textFieldRules"
          value="Russia"
        ></v-autocomplete>
      </v-col>
      <v-col></v-col>
    </v-row>
    <template v-if="!isRussian">
      <v-row>
        <v-col>
          <v-text-field
            name="latinLastName"
            label="Фамилия на латинице"
            :rules="latinCharsRule"
          />
        </v-col>
        <v-col>
          <v-text-field
            name="latinFirstName"
            label="Имя на латинице"
            :rules="latinCharsRule"
          />
        </v-col>
      </v-row>
    </template>
    <v-row>
      <v-col>
        <v-text-field
          name="passportNumber"
          label="Номер паспорта"
          :rules="passportNumberFieldRules"
        />
      </v-col>
      <template v-if="isRussian">
        <v-col>
          <v-text-field
            name="passportSeries"
            label="Серия паспорта"
            :rules="passportSeriesFieldRules"
          />
        </v-col>
        <v-col>
          <v-menu
            ref="menu"
            v-model="menu"
            :close-on-content-click="false"
            transition="scale-transition"
            offset-y
            min-width="auto"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                name="passportIssueDate"
                v-model="issueDate"
                label="Дата выдачи"
                placeholder="дд.мм.гг"
                prepend-icon="mdi-calendar"
                readonly
                v-bind="attrs"
                v-on="on"
                :rules="dateRules"
              ></v-text-field>
            </template>
            <v-date-picker
              v-model="issueDate"
              :active-picker.sync="activePicker"
              :max="
                new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
                  .toISOString()
                  .substr(0, 10)
              "
              min="1980-01-01"
              @change="save"
              locale="ru"
            ></v-date-picker>
          </v-menu>
        </v-col>
      </template>
      <template v-else>
        <v-col>
          <v-select
            name="passportIssueCountry"
            label="Страна выдачи"
            :items="issueCountries"
            v-model="issueCountry"
            :rules="textFieldRules"
          ></v-select>
        </v-col>
        <v-col>
          <v-select
            name="passportType"
            label="Тип паспорта"
            :items="passportTypes"
            v-model="passportType"
            :rules="textFieldRules"
          ></v-select>
        </v-col>
      </template>
    </v-row>
    <v-row>
      <v-col>
        <p>Меняли ли фамилию или имя?</p>
        <v-radio-group name="nameChanged" mandatory @change="nameChangeEvent">
          <v-radio label="Нет" value="false"></v-radio>
          <v-radio label="Да" value="true"></v-radio>
        </v-radio-group>
      </v-col>
    </v-row>
    <v-row v-if="nameChanged">
      <v-col>
        <v-text-field
          name="newLastName"
          label="Фамилия"
          outlined
          :rules="onlyRussianChars"
          v-model="newLastName"
        />
      </v-col>
      <v-col>
        <v-text-field
          name="newFirstName"
          label="Имя"
          outlined
          :rules="onlyRussianChars"
          v-model="newFirstName"
        />
      </v-col>
    </v-row>
  </v-col>
</template>

<script>
import citizenships from "../assets/data/citizenships.json";
import passportTypes from "../assets/data/passport-types.json";
import throttle from "../utils/utils";
import validator from "validator";
import Fuse from "fuse.js";

export default {
  data: () => {
    return {
      textFieldRules: [
        (value) => {
          if (value === (undefined || null) || value?.length === 0) {
            return "Обязательное поле";
          } else {
            return true;
          }
        },
      ],
      dateRules: [
        (value) => {
          if (value === undefined || value === null || value?.length === 0) {
            return "Введите дату выдачи паспорта";
          } else {
            return true;
          }
        },
      ],
      passportSeriesFieldRules: [
        (value) => {
          if (value === undefined || value === null || value?.length === 0) {
            return "Введите серию паспорта";
          }
        },
        (value) => {
          if (value !== undefined) {
            return value.length === 4 && !Number.isNaN(Number(value))
              ? true
              : "Серия паспорта должна содержать 4 цифры.";
          }
        },
      ],
      onlyRussianChars: [
        (value) => !!value || "Обязательное поле",
        (value) => {
          if (value === undefined) return true;
          return validator.isAlpha(value, "ru-RU")
            ? true
            : "Допустимы только русские буквы.";
        },
      ],
      latinCharsRule: [
        (value) => {
          if (value?.length === 0 || value === undefined) {
            return "Обязательное поле";
          } else {
            return validator.isAlpha(value, "en-US")
              ? true
              : "Только латинские буквы.";
          }
        },
      ],
      activePicker: null,
      menu: false,
      citizenships: citizenships,
      filteredNationalities: null,
      isRussian: true,
      issueDate: null,
      issueCountry: null,
      passportType: null,
      nameChanged: false,
      newFirstName: "",
      newLastName: "",
    };
  },
  created() {
    this.filteredNationalities = this.citizenshipNationalities;
  },
  methods: {
    displayFiltered(inputVal) {
      if (inputVal === null || inputVal.length === 0) {
        this.filteredNationalities = this.citizenshipNationalities;
        return;
      }
      const options = {
        includeScore: true,
        keys: ["nationality"],
      };
      const fuse = new Fuse(this.citizenships, options);
      const res = fuse.search(inputVal);
      this.filteredNationalities = res.map(({ item }) => {
        return item.nationality;
      });
    },
    passportFieldsController(val) {
      if (val === "Russia") {
        this.isRussian = true;
      } else {
        this.isRussian = false;
      }
    },
    nameChangeEvent(val) {
      if (val === "false") {
        this.nameChanged = false;
      } else if (val === "true") {
        this.nameChanged = true;
      } else {
        return;
      }
    },
    save(date) {
      this.$refs.menu.save(date);
    },
  },
  computed: {
    citizenshipNationalities: function () {
      return citizenships.map((item) => item.nationality);
    },
    throttledDisplayFiltered: function () {
      return throttle(this.displayFiltered, 200);
    },
    issueCountries: function () {
      return citizenships.map((item) => item.flag);
    },
    passportTypes: function () {
      return passportTypes.map((passport) => passport.type);
    },
    passportNumberFieldRules: function () {
      return [
        (value) => {
          if (value?.length === 0 || value === undefined) {
            return "Обязательное поле";
          } else {
            return true;
          }
        },
        (value) => {
          if (this.isRussian) {
            return (
              value?.length === 6 &&
              !Number.isNaN(
                Number(value)
                  ? true
                  : "Номер паспорта РФ должен содержать 6 цифр"
              )
            );
          } else {
            const isNum = !Number.isNaN(Number(value));
            return isNum ? true : "Номер может содержать только цифры.";
          }
        },
      ];
    },
  },
  watch: {
    menu(val) {
      val && setTimeout(() => (this.activePicker = "YEAR"));
    },
  },
};
</script>
