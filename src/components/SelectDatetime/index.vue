<template>
  <div class="SelectDatetime">
    <v-row align="center">
      <v-col cols="12" md="4" sm="4">
        <v-select
          v-model="year"
          :items="yearList"
          :label="$t('label.year')"
          solo
          @change="onSelectYear"
        />
      </v-col>
      <v-col cols="12" md="4" sm="4">
        <v-select
          v-model="month"
          :items="listMonthName"
          :disabled="disabledMonth"
          :label="$t('label.month')"
          item-value="value"
          item-text="text"
          solo
          @change="onSelectMonth"
        />
      </v-col>
      <v-col cols="12" md="4" sm="4">
        <v-select
          v-model="days"
          :items="dayList"
          :disabled="disabledDays"
          :label="$t('label.date')"
          solo
          @change="onSelectDate"
        />
      </v-col>
    </v-row>
  </div>
</template>

<script>
import { listYear, listMonthName, listDays } from '@/utils/constantVariable'

export default {
  name: 'SelectDatetime',
  props: {
    datetime: {
      type: String,
      default: ''
    },
    formateDate: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      year: '',
      month: '',
      days: '',
      date: [],
      finalDate: '',
      yearList: null,
      listMonthName: listMonthName,
      dayList: null,
      disabledMonth: true,
      disabledDays: true
    }
  },
  async mounted() {
    this.yearList = await this.listYear()
    this.dayList = await this.listDays()
    this.date = await this.datetime.split('/')
    if (this.date[1]) {
      this.disabledMonth = false
    }
    if (this.date[2]) {
      this.disabledDays = false
    }
    this.year = parseInt(this.date[0])
    this.month = parseInt(this.date[1])
    this.days = parseInt(this.date[2])
  },
  methods: {
    listYear,
    listDays,
    onSelectYear(value) {
      this.disabledMonth = false
      if (this.date.length >= 3) {
        this.date[0] = value
        this.finalDate = `${this.date[0]}/${this.date[1]}/${this.date[2]}`
        this.$emit('update:dateTime', this.finalDate)
      } else {
        this.date[0] = value
      }
    },
    onSelectMonth(value) {
      this.disabledDays = false
      if (this.date.length >= 3) {
        this.date[1] = value
        this.finalDate = `${this.date[0]}/${this.date[1]}/${this.date[2]}`
        this.$emit('update:dateTime', this.finalDate)
      } else {
        this.date[1] = value
      }
    },
    onSelectDate(value) {
      if (this.date.length >= 3) {
        this.date[2] = value
        this.finalDate = `${this.date[0]}/${this.date[1]}/${this.date[2]}`
        this.$emit('update:dateTime', this.finalDate)
      } else {
        if (this.date.length >= 2) {
          this.date[2] = value
          this.finalDate = `${this.date[0]}/${this.date[1]}/${this.date[2]}`
          this.$emit('update:dateTime', this.finalDate)
        }
      }
    }
  }
}
</script>
