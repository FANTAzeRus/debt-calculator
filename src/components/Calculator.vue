<template>
  <div class="container">
    <div class="calculator">
      <div class="start-form" v-if="state.showStartForm">
        <label for="startDate">Дата начала просрочки:</label>
        <input type="text" id="startDate" v-model="state.startDate" />

        <label for="startDebt">Сумма задолженности:</label>
        <input type="text" id="startDebt" v-model="state.startDebt" />

        <button class="btn" :disabled="!canCalculate" @click="calculateData">
          Рассчитать
        </button>
      </div>

      <div class="calculate-block" v-else>
        <div class="list-item list-header">
          <div class="col col-header">Дата начала просрочки</div>
          <div class="col col-header">Количество дней просрочки</div>
          <div class="col col-header">Начальная задолженность</div>
          <div class="col col-header calc">Расчет</div>
          <div class="col col-header">Пени</div>
          <div class="col col-header">Задолженность на сегодняшний день</div>
        </div>

        <div
          class="list-item"
          v-for="(line, idx) in items"
          :key="`line_${idx}`"
        >
          <div class="col">{{ line.date }}</div>
          <div class="col">{{ line.days }}</div>
          <div class="col">
            <input type="text" v-model="state.lines[idx].debt" />
          </div>
          <div class="col calc">{{ line.calc }}</div>
          <div class="col">{{ formatedRub(line.fine) }}</div>
          <div class="col">{{ formatedRub(line.summ) }}</div>
        </div>

        <div class="total">
          <div class="label">Итого:</div>
          <div class="value">{{ formatedRub(total) }}</div>
        </div>

        <div class="total">
          <div class="label"></div>
          <div class="value">
            <button class="btn" @click="newCalculate">Новый расчет</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import moment from "moment";
import { computed, reactive } from "vue";

export default {
  name: "DebtCalculator",
  setup() {
    const state = reactive({
      format: "DD.MM.YYYY",
      showStartForm: true,
      startDate: null,
      startDebt: 0,
      lines: [],
    });

    const lineStructure = {
      date1: null,
      date2: null,
      debt: null,
      fine: null,
      days: null,
      summ: null,
    };

    const fillDatesFromNow = (start) => {
      const sd = moment(start, state.format, true);
      const dn = moment();
      const months = dn.diff(sd, "months");

      let ret = [];
      let first = true;

      for (let i = 0; i <= months; i++) {
        let calc = sd.add(first ? 0 : 1, "month");
        if (calc < dn) {
          ret.push(calc.format(state.format));
        }
        first = false;
      }

      return ret;
    };

    const calculateData = () => {
      const dates = fillDatesFromNow(state.startDate);
      dates.forEach((d) => {
        state.lines.push(
          Object.assign(
            {},
            {
              date1: moment(d, state.format, true),
              date2: moment(),
              debt: Number(state.startDebt),
              fine: null,
              days: null,
              summ: null,
            }
          )
        );

        if (state.lines.length) {
          state.showStartForm = false;
        }
      });

      console.log(state.lines);
    };

    const canCalculate = computed(() => {
      let date = moment(state.startDate, state.format, true);

      return date.isValid() && state.startDebt > 0;
    });

    const items = computed(() =>
      state.lines.map((line) => {
        const k1 = 1 / 150;
        const p1 = 0.0425;
        const days = line.date2.diff(line.date1, "days");
        const fine = Number(line.debt) * days * k1 * p1;
        const summ = Number(line.debt) + Number(fine);
        const calc = `${formatedRub(line.debt)} x ${days} x 1/50 x 4.25%`;

        return {
          date: line.date1.format(state.format),
          debt: line.debt,
          days,
          fine,
          summ,
          calc,
        };
      })
    );

    const formatedRub = (val) => {
      return `${new Intl.NumberFormat("ru-RU", {
        maximumFractionDigits: 2,
        minimumFractionDigits: 2,
      }).format(val)} руб.`;
    };

    const total = computed(() => {
      if (!items.value.length) return 0;

      return items.value.reduce((sum, val) => {
        return Number(sum) + Number(val.summ);
      }, 0);
    });

    const newCalculate = () => {
      state.lines = [];
      state.showStartForm = true;
      Object.assign(state, { startDate: null, startDebt: 0 });
    };

    return {
      state,
      canCalculate,
      calculateData,
      items,
      formatedRub,
      total,
      newCalculate,
    };
  },
};
</script>