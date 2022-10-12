<template>
  <v-text-field
    class="rounded-lg"
    ref="input"
    dense
    hide-spin-buttons
    :background-color="bgColor"
    outlined
    :value="value"
    :label="label"
    :placeholder="placeholder"
    :height="height"
    :rules="enableRules"
    @input="updateValue($event)"
    @keydown.enter.prevent
    :prefix="prefix"
    :type="type"
    :error="error"
    :error-messages="errorMessage"
    :append-icon="icon"
    :min="min"
    @click:append="showPassword = !showPassword"
    :disabled="disabled"
    :readonly="readonly"
    :messages="message"
    :autofocus="autofocus"
    :color="color"
  />
</template>

<script>
/** Компонент ввода текстового поля, также подходит для полей пароля */
import { mapState } from "vuex";

export default {
  name: "TextInput",
  props: {
    /** Значение поля, обязательные параметр*/
    value: {
      required: true,
    },
    /** label поля*/
    label: {
      type: String,
    },
    /** placeholder поля*/
    placeholder: {
      type: String,
    },
    /** Высота поля*/
    height: {
      type: String,
      default: "42",
    },
    /**
     * Правила валидации
     * @values required, min, email, phone, string
     */
    rules: {
      type: Array,
    },
    /** Флаг для обозначения, что данное поле для пароля*/
    isPassword: {
      type: Boolean,
      default: false,
    },
    /** Массив с двумя паролями для их сравнения*/
    passComparison: {
      type: Array,
    },
    /** Задает цвет фона поля*/
    bgColor: {
      type: String,
    },
    /** Поле для передачи текста сообщения об ошибке*/
    errorMessage: {
      type: String,
    },
    /** Флаг, вызывает подсветку ошибки поля */
    error: {
      type: Boolean,
    },
    /** Флаг для обозначения, что данное поле числовое*/
    isNumber: {
      type: Boolean,
      default: false,
    },
    /** Максимальное значение для числового ввода */
    max: {
      type: String,
    },
    /** Устанавливает минимальное значение для числового поля*/
    min: {
      type: String,
    },
    /** Флаг блокировки поля*/
    disabled: {
      type: Boolean,
      default: false,
    },
    /** Флаг - поле только для чтения (своего рода блокировка)*/
    readonly: {
      type: Boolean,
      default: false,
    },
    /** Символ в начале поля ввода */
    prefix: {
      type: String,
    },
    /** Устанавливает сообщение под поле */
    message: {
      type: String,
    },
    /** Устанавливает фокус на поле */
    autofocus: {
      type: Boolean,
    },
    /** Задает цвет поля */
    color: {
      type: String,
    },
    /** Флаг, устанавливает спец класс на корневой input компонента */
    yandex: {
      type: Boolean,
    },
  },

  data() {
    return {
      /** Флаг, показывать пароль или нет*/
      showPassword: false,
      /** в случае если поле-пароль, устанавливается кнопка-иконка для срытия/показа пароля*/
      icon: "",
      /** Тип поля, текстовый или пароль*/
      type: "text",
      /** Правила валидации*/
      validateRules: {
        required: (v) => v !== "" || "Обязательное поле",
        min: (v) => v.length >= 6 || "Минимум 6 символов",
        max: (v) => v <= +this.max || `Максимальное значение: ${this.max}`,
        password: (v1, v2) => v1 === v2 || "Пароли должны совпадать",
        email: (v) => {
          const pattern =
            /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
          return pattern.test(v) || "Некорректный e-mail";
        },
        phone: (v) => {
          const pattern = /\(\d{3}\)\d{3}-\d{2}-\d{2}/;
          return pattern.test(v) || "Некорректный номер телефона";
        },
        string: (v) => {
          const pattern = /^[а-яА-ЯёЁ ]+$/g;
          return pattern.test(v) || "Только русские буквы";
        },
        number: (v) => {
          const pattern = /^\d+$/;
          return pattern.test(v) || "Только цифры";
        },
        numberMin: (v) => {
          const pattern = /^[0-9]\d*$/;
          return pattern.test(v) || "Не может быть отрицательным";
        },
        promoCode: (v) => {
          const pattern = /^[A-Za-z0-9]+/;
          return pattern.test(v) || "Только цифры и английские буквы";
        },
        useBonus: (v) =>
          v <= this.user.bonuses || "Указанное значение больше доступного",
      },
    };
  },

  methods: {
    /**
     * Обновляет значение поля
     * @param val актуальное значение
     */
    updateValue(val) {
      this.$emit("input", val);
    },
  },

  computed: {
    ...mapState("user", ["user"]),

    /**
     * Применение правил валидации к полю
     * @returns {*[]} возвращает массив функций
     */
    enableRules() {
      let rules = [];

      Object.keys(this.validateRules).forEach((rule) => {
        if (this.rules !== undefined && this.rules.includes(rule)) {
          rules.push(this.validateRules[rule]);
        }
      });

      if (this.isNumber && this.max !== undefined) {
        rules.push(this.validateRules.max(+this.value));
      }

      if (this.passComparison !== undefined) {
        rules.push(
          this.validateRules.password(
            this.passComparison[0],
            this.passComparison[1]
          )
        );
      }

      return rules;
    },
  },

  mounted() {
    /** Если стоит флаг yandex, то устанавливает класс метрики на корневой input */
    if (this.yandex) {
      this.$refs.input.$el
        .querySelector("input")
        .classList.add("ym-disable-keys");
    }

    /** В случае если поле - пароль, задает тип поля и устанавливает картинку*/
    if (this.isPassword) {
      this.icon = "mdi-eye-off";
      this.type = "password";
    }

    /** В случае если поле числовое, задает тип поля*/
    if (this.isNumber) {
      this.type = "number";
    }
  },

  watch: {
    /** Смотрит за изменение флага и отображает или скрывает пароль с изменением иконки*/
    showPassword() {
      if (this.isPassword) {
        if (this.showPassword) {
          this.icon = "mdi-eye";
          this.type = "text";
        } else {
          this.icon = "mdi-eye-off";
          this.type = "password";
        }
      }
    },
  },
};
</script>
