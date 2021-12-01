<template>
  <v-container>
    <v-row>
      <v-col>
        <v-form ref="form">
          <v-row>
            <v-col>
              <h3>Личные данные</h3>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <v-text-field
                name="lastName"
                label="Фамилия"
                outlined
                :rules="onlyRussianChars"
                v-model="formData.lastName"
              />
            </v-col>
            <v-col>
              <v-text-field
                name="firstName"
                label="Имя"
                autocomplete="name"
                outlined
                :rules="onlyRussianChars"
                v-model="formData.firstName"
              />
            </v-col>
            <v-col>
              <v-text-field
                name="middleName"
                label="Отчество"
                outlined
                :rules="onlyRussianChars"
                v-model="formData.middleName"
              />
            </v-col>
          </v-row>
          <v-row>
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
                    name="birthDate"
                    v-model="formData.birthDate"
                    label="Дата рождения"
                    placeholder="дд.мм.гг"
                    prepend-icon="mdi-calendar"
                    readonly
                    v-bind="attrs"
                    v-on="on"
                    :rules="textFieldRules"
                  ></v-text-field>
                </template>
                <v-date-picker
                  v-model="formData.birthDate"
                  :active-picker.sync="activePicker"
                  :max="
                    new Date(
                      Date.now() - new Date().getTimezoneOffset() * 60000
                    )
                      .toISOString()
                      .substr(0, 10)
                  "
                  min="1950-01-01"
                  @change="save"
                  locale="ru"
                ></v-date-picker>
              </v-menu>
            </v-col>
            <v-col></v-col>
          </v-row>
          <v-row>
            <v-col>
              <v-text-field
                name="email"
                label="Email"
                type="email"
                autocomplete="email"
                v-model="formData.email"
                :rules="emailFieldRules"
              />
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <p>Пол</p>
              <v-radio-group name="gender" v-model="formData.gender" mandatory>
                <v-radio label="Мужской" value="male"></v-radio>
                <v-radio label="Женский" value="female"></v-radio>
              </v-radio-group>
            </v-col>
          </v-row>
          <v-row>
            <passport-data></passport-data>
          </v-row>
          <v-row>
            <v-col>
              <v-btn color="primary" @click.prevent="submitForm" type="submit">
                Отправить
              </v-btn>
            </v-col>
          </v-row>
        </v-form>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import PassportData from "./PassportData";
import validator from "validator";
export default {
  components: {
    PassportData,
  },
  data: () => {
    return {
      textFieldRules: [(value) => !!value || "Обязательное поле"],
      emailFieldRules: [
        (value) => !!value || "Обязательное поле",
        (value) => {
          if (value === undefined) {
            return true;
          }
          const emailRegExp = new RegExp(/^\S+@\S+\.\S+$/);
          return value.match(emailRegExp) ? true : "Неверный формат почты";
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
      activePicker: null,
      menu: false,
      formData: {
        firstName: "",
        lastName: "",
        middleName: "",
        birthDate: null,
        email: "",
        gender: "",
      },
    };
  },
  methods: {
    save(date) {
      this.$refs.menu.save(date);
    },
    submitForm() {
      console.log("form submit event");
      const validationRes = this.$refs.form.validate();
      console.log("VALIDATION RESULT", validationRes);
      if (validationRes) {
        console.log("Validation passed");
        const formData = new FormData(this.$refs.form.$el);
        const readyData = {};
        for (const [key, value] of formData.entries()) {
          readyData[key] = value;
        }
        // sendReadyData(readyData);
      }
    },
  },
  watch: {
    menu(val) {
      val && setTimeout(() => (this.activePicker = "YEAR"));
    },
  },
};
</script>

<style scoped></style>
