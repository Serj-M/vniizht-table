<template>
  <v-card>
    <br>
    <v-card-title>
      <h2>Таблица с данными по вагонам</h2>
    </v-card-title>
    <v-data-table
      :headers="headers"
      :items="tableData"
      :search="search"
      class="elevation-1"
      :footer-props="{
          itemsPerPageText: 'Строк на странице'
      }"
    >
      <template v-slot:top>
        <v-toolbar class="my-toolbar" flat color="white">
          <v-container class="my-container" fluid>
            <v-row align="center">
              <v-col class="d-flex" cols="12" sm="6">
                <v-select
                  v-model="selectedColumn"
                  :items="columns"
                  label="Выберите столбец"
                ></v-select>
              </v-col>
              <v-col class="d-flex" cols="12" sm="6">
                <v-text-field
                  v-if="selectedColumn === 'dateInMilliseconds'"
                  v-model="search"
                  append-icon="mdi-magnify"
                  label="Поиск по дате ( вводите в формате миллисекунд ;=) )"
                  single-line
                ></v-text-field>
                <v-text-field
                  v-else
                  v-model="search"
                  :rules="[rules.simpleSearch]"
                  append-icon="mdi-magnify"
                  label="Поиск"
                  single-line
                ></v-text-field>
              </v-col>
            </v-row>
          </v-container>
          <v-dialog v-model="dialog" max-width="600px">
            <v-card>
              <v-card-title>
                <span class="headline">Редактирование данных по вагону</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field disabled v-bind:value="editedItem.ordNumber" label="№ п/п"></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field disabled v-bind:value="editedItem.carNumber" label="Номер вагона"></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field disabled v-bind:value="editedItem.trainIndex" label="Индекс поезда"></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.trainNumber"
                        :rules="[rules.trainNumberValid]"
                        v-mask="rules.trainNumberMask"
                        label="Номер поезда"
                        counter
                        maxlength="4"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field disabled v-bind:value="editedItem.carStatus" label="Статус"></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        id="idDT"
                        v-model="editedItem.lastOperDt"
                        :rules="[rules.lastOperDtValid]"
                        v-mask="rules.lastOperDtMask"
                        label="Дата-время операции"
                        counter
                        maxlength="16"
                        :key="editedItem.ordNumber"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.invoiceNumber"
                        :rules="[rules.invoiceNumberValid]"
                        v-mask="rules.invoiceNumberMask"
                        label="№ накладной (обязательно)"
                        counter
                        maxlength="8"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.invoiceId"
                        :rules="[rules.invoiceIdValid]"
                        v-mask="rules.invoiceIdMask"
                        label="ИД накладной"
                        counter
                        maxlength="9"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field disabled v-bind:value="editedItem.stateId" label="stateId"></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">Отмена</v-btn>
                <v-btn color="blue darken-1" text @click="save" :disabled="!dataIsValid">ОК</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:item.dateInMilliseconds="{ item }">
        <!-- сортировка даты по dateInMilliseconds, а отображается формат ДД.ММ.ГГГГ ЧЧ:мм -->
        {{ item.lastOperDt }}
      </template>
      <template v-slot:item.action="{ item }">
        <v-icon
          small
          class="mr-2"
          @click="editItem(item)"
        >
          fa-edit
        </v-icon>
      </template>
      <template v-slot:no-data>
        <h2>Нет данных</h2>
      </template>
    </v-data-table>
  </v-card>
</template>

<script>
import Vue from 'vue'
import { mask } from 'vue-the-mask'
import moment from 'moment'
Vue.prototype.$moment = moment
let toggleForSearch = false

// меняет местами месяц и день в формате 'DD.MM.YYYY HH:mm', для использования в Date.parse(fixValue)
function fixDate (dt) {
  const arrData = dt.split('.')
  const temp = arrData[0]
  arrData[0] = arrData[1]
  arrData[1] = temp
  const strData = arrData.join('.')
  return strData
}

export default {
  // добавление директивы mask из плагина vue-the-mask (маска для полей ввода)
  directives: {
    mask
  },

  data: () => ({
    dialog: false,
    tableData: [], // массив данных для таблицы
    search: '',
    selectedColumn: null,

    // формирование выпадающего списка для фильтрации поиска
    columns: [
      { text: '№ п/п', value: 'ordNumber' },
      { text: 'Номер вагона', value: 'carNumber' },
      { text: 'Индекс поезда', value: 'trainIndex' },
      { text: 'Номер поезда', value: 'trainNumber' },
      { text: 'Статус', value: 'carStatus' },
      { text: 'Дата-время операции', value: 'dateInMilliseconds' },
      { text: '№ накладной', value: 'invoiceNumber' },
      { text: 'ИД накладной', value: 'invoiceId' },
      { text: 'stateId', value: 'stateId' }
    ],

    // формирование заголовка таблицы
    headers: [
      { text: '№ п/п', value: 'ordNumber', align: 'center', filterable: false },
      { text: 'Номер вагона', value: 'carNumber', align: 'center', filterable: false },
      { text: 'Индекс поезда', value: 'trainIndex', align: 'center', filterable: false },
      { text: 'Номер поезда', value: 'trainNumber', align: 'center', filterable: false },
      { text: 'Статус', value: 'carStatus', align: 'center', filterable: false },
      { text: 'Дата-время операции', value: 'dateInMilliseconds', align: 'center', filterable: false },
      { text: '№ накладной', value: 'invoiceNumber', align: 'center', filterable: false },
      { text: 'ИД накладной', value: 'invoiceId', align: 'center', filterable: false },
      { text: 'stateId', value: 'stateId', align: 'center', filterable: false },
      { text: '', value: 'action', sortable: false, align: 'center', filterable: false }
    ],

    // свойства формы редактирования со значением по умолчанию
    editedItem: {
      // ordNumber: 0,
      // carNumber: 0,
      // trainIndex: 0,
      trainNumber: 0,
      // carStatus: '',
      lastOperDt: '',
      dateInMilliseconds: 0,
      invoiceNumber: '',
      invoiceId: 0
      // stateId: 0
    },

    // правила валидации формы
    rules: {
      // правила для Номера поезда
      trainNumberMask: '####',
      trainNumberValid: value => {
        if (!value) {
          return true
        }
        const pattern = /[0-9]{4}/
        return pattern.test(value) || '4 цифры'
      },

      // правила для № накладной (Обязательное поле для заполнения)
      invoiceNumberMask: 'ЭО######',
      invoiceNumberValid: value => {
        const pattern = /ЭО[0-9]{6}/
        return pattern.test(value) || 'формат ЭО123456'
      },

      // правила для Даты-время
      lastOperDtMask: '##.##.#### ##:##',
      lastOperDtValid: value => {
        if (value === '-' || !value) {
          return true
        }
        if (moment(value, 'DD.MM.YYYY HH:mm').isValid()) {
          const currentTime = new Date().toString()
          const currentTimeTrueFormat = Vue.prototype.$moment(currentTime).format('DD.MM.YYYY HH:mm')
          const fixCurrentTimeTrueFormat = fixDate(currentTimeTrueFormat)
          const currentTimeInMilliseconds = Date.parse(fixCurrentTimeTrueFormat)
          const fixValue = fixDate(value)
          const valueInMilliseconds = Date.parse(fixValue)
          if (valueInMilliseconds > currentTimeInMilliseconds) {
            return 'дата должна быть до текущего момента'
          } else {
            return true
          }
        } else {
          return 'формат дд.мм.гггг чч:мм'
        }
      },

      // проверка для ИД накладной
      invoiceIdMask: '#########',
      invoiceIdValid: value => {
        if (!value) {
          return true
        }
        const pattern = /[0-9]{9}/
        return pattern.test(value) || '9 цифр'
      },

      // проверка выбран ли столбец для поиска
      simpleSearch: value => {
        if (value && !toggleForSearch) {
          return 'Выберите столбец для поиска'
        }
        return true
      }
    }
  }),

  // отслеживание изменений состояния валидации формы
  computed: {
    // проверка валидности введенного Номер поезда
    trainNumberIsValid () {
      if (this.editedItem.trainNumber === null || this.editedItem.trainNumber === '') {
        return true
      }
      return this.rules.trainNumberValid(this.editedItem.trainNumber) !== '4 цифры'
    },

    // проверка валидности введенного № накладной
    invoiceNumberIsValid () {
      if (this.rules.invoiceNumberValid(this.editedItem.invoiceNumber) === true) {
        return true
      }
      return false
    },

    // проверка валидности введенной Даты-время
    lastOperDtIsValid () {
      if (this.editedItem.lastOperDt === '-' || !this.editedItem.lastOperDt) {
        // костыль с черточкой если пустая дата. кнопка ОК активна с незаполненным полем
        return true
      }
      if (this.rules.lastOperDtValid(this.editedItem.lastOperDt) === true && String(this.editedItem.lastOperDt).length === 16) {
        return true
      }
      return false
    },

    // проверка валидности введенной ИД накладной
    invoiceIdIsValid () {
      if (this.rules.invoiceIdValid(this.editedItem.invoiceId) === true) {
        return true
      }
      return false
    },

    // условие кнопки сохранение (активна/не активна)
    dataIsValid () {
      return this.trainNumberIsValid && this.invoiceNumberIsValid && this.lastOperDtIsValid && this.invoiceIdIsValid
    }
  },

  watch: {
    dialog (val) {
      val || this.close()
    },

    // отслеживает изменения поля selectedColumn подключенного к селекту
    // и включает фильтрацию в выбранном столбце для дальнейшего поиска
    selectedColumn (val) {
      const selected = this.headers.find(item => item.value === val)
      selected.filterable = true
      this.search = ''
      toggleForSearch = true
    },

    // НЕ ЗАКОНЧЕНА! отслеживает изменения поля search если фильтрация настроена на Дату-время
    search (val) {
    //   this.tableData = require('../table-data.json')
    //   this.dateTime(this.tableData)
    //   if (this.selectedColumn === 'dateInMilliseconds') {
    //     // попытка с поиском в строке 'DD.MM.YYYY HH:mm'
    //     // this.tableData = this.tableData.filter((item) => item.lastOperDt.includes(val))
    //     // console.log(this.tableData)
    //     // попытка с миллисекундами
    //     if ((this.rules.lastOperDtValid(val) === true && String(val).length === 16)) {
    //       const searchDate = fixDate(val)
    //       const searchDateInMilliseconds = Date.parse(searchDate)
    //       this.tableData = this.tableData.filter((item) => item.dateInMilliseconds === searchDateInMilliseconds)
    //       // console.log(this.tableData)
    //       // return searchDateInMilliseconds
    //     }
    //   }
    }
  },

  // хук жизненного цикла - выполнение кода сразу после создания экземпляра
  created () {
    this.initialize()
  },

  methods: {
    // данные из файла table-data.json
    initialize () {
      this.tableData = require('../table-data.json')
      this.dateTime(this.tableData)
    },

    // получение строки для редактирования в модальном окне
    editItem (item) {
      this.editedIndex = this.tableData.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    // отмена редактирования и закрытие модального окна
    close () {
      this.dialog = false
    },

    // сохранение редактирования и закрытие модального окна
    save () {
      // костыль для отмены повторного преобразования формата Даты-время при обновлении состояния в жизненном цикле
      if (this.editedItem.lastOperDt === '-' || !this.editedItem.lastOperDt) {
        // для сортировки по дате
        this.editedItem.dateInMilliseconds = 0
      } else {
        const dateSave = fixDate(this.editedItem.lastOperDt)
        // отредактированную дату перевожу в миллисекунды
        this.editedItem.dateInMilliseconds = Date.parse(dateSave)
      }
      Object.assign(this.tableData[this.editedIndex], this.editedItem)
      this.close()
    },

    // ф-ция настройки формата времени
    dateTime (dt) {
      dt = dt.map(function (item) {
        // костыль для отмены повторного преобразования формата Даты-время при обновлении состояния в жизненном цикле
        if (!isNaN(item.dateInMilliseconds)) {
          return item
        }
        if (item.lastOperDt === null) {
          item.lastOperDt = '-'
          item.dateInMilliseconds = 0
          return item
        } else {
          // убираю Z в конце строки '2019-07-28T00:30:00.000Z', что-бы не учитывать UTC(GMT+03)
          const tempDate = item.lastOperDt.substr(0, 22)
          // добавляю в объект св-во со значением даты в миллисекундах для дальнейшей сортировки
          item.dateInMilliseconds = Date.parse(tempDate)
          // преобразование формата Даты-время. PS: HH - это 24-х часовой формат времени
          item.lastOperDt = Vue.prototype.$moment(tempDate).format('DD.MM.YYYY HH:mm')
          return item
        }
      })
    }
  }
}
</script>

<style>
.my-container {
  margin-top: 50px;
  margin-bottom: 30px
}

@media screen and (max-width:500px) {
  .my-toolbar {
    margin-top: 50px;
    margin-bottom: 50px
  }
}
</style>
