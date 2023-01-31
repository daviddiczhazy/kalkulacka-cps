<script setup lang="ts">
import { ref, watch } from 'vue'
// eslint-disable-next-line no-undef
const props = defineProps<{
  msg?: string
}>()
// types
type variantOfInsurance = 'short' | 'long'
type numberOfPersons = 1 | 2 | 3
// constants
const variantIns = ref<variantOfInsurance>('short')
const startOfIns = ref<string>()
const endOfIns = ref<string>()
const packetIns = ref<number>(0)
const stornoIns = ref<boolean>(false)
const sportIns = ref<boolean>(false)
const personsIns = ref<numberOfPersons>(1)

const checkFull = ref<boolean>(false)
const result = ref<string>()
const errorMessage = ref<string>('')

const startDateEmpty = ref<boolean>(false)
const endDateEmpty = ref<boolean>(false)

watch(variantIns, () => {
  result.value = ''
  startOfIns.value = ''
  endOfIns.value = ''
  if (variantIns.value === 'long') {
    checkFull.value = true
  } else {
    checkFull.value = false
  }
})
// Function for calculation full year Insurance
const getFullInsValue = (
  num: number[],
  storno: number,
  sport: number
): number => {
  const sum = num[packetIns.value] * storno * sport * personsIns.value
  return Math.round((sum + Number.EPSILON) * 100) / 100
}
// Function for calculation shor Insurance according days
const getShortInsValue = (
  num: number[],
  storno: number,
  sport: number,
  days = 1
): number => {
  const sum = num[packetIns.value] * storno * sport * personsIns.value * days
  return Math.round((sum + Number.EPSILON) * 100) / 100
}
// Function which return number of day in number format
const getDayNumber = (InsuranceDay: string): number => {
  let dateCheck = new Date(InsuranceDay)
  const resultDay = dateCheck.getTime() / (1000 * 3600 * 24)
  return resultDay
}

const checkInputs = (): boolean => {
  if (!startOfIns.value) {
    startDateEmpty.value = true
  } else {
    startDateEmpty.value = false
  }

  if (!checkFull.value) {
    if (!endOfIns.value) {
      endDateEmpty.value = true
    } else {
      endDateEmpty.value = false
    }
  }

  return true
}
// The main function for checking and calculating sum of insurance
const doCalc = () => {
  errorMessage.value = ''
  if (checkInputs()) {
    if (checkFull.value) {
      const insSum: number[] = [39, 49, 59]
      const stornoAdd = stornoIns.value ? 1.2 : 1
      const sportAdd = sportIns.value ? 1.1 : 1
      const fullSum = getFullInsValue(insSum, stornoAdd, sportAdd)
      if (startOfIns.value) {
        result.value = fullSum.toString().replace('.', ',') + ' €'
      } else {
        return (errorMessage.value =
          'CHYBA *potrebné vyplniť všetky povinné polia')
      }
    } else {
      const InsuranceDayCount =
        getDayNumber(endOfIns.value as string) -
        getDayNumber(startOfIns.value as string)

      if (InsuranceDayCount < 0) {
        return (errorMessage.value = 'CHYBA: zly výber rozsahu datumov')
      } else if (InsuranceDayCount === 0) {
        return (errorMessage.value = 'CHYBA: Vybrali ste 0 dni')
      } else if (InsuranceDayCount > 356) {
        return (errorMessage.value =
          'Vybrali ste viac ako jeden rok, vyberte si radsej celorocne poistenie')
      } else if (!InsuranceDayCount) {
        return (errorMessage.value =
          'CHYBA *potrebné vyplniť všetky povinné polia')
      } else {
        const insSum: number[] = [1.2, 1.8, 2.4]
        const stornoAdd = stornoIns.value ? 1.5 : 1
        const sportAdd = sportIns.value ? 1.3 : 1
        const shortSum = getShortInsValue(
          insSum,
          stornoAdd,
          sportAdd,
          InsuranceDayCount
        )
        result.value = shortSum.toString().replace('.', ',') + ' €'
      }
    }
  } else return
}
</script>

<template>
  <div class="hello">
    <h1>Poistná kalkulačka</h1>
    <p>{{ props.msg }}</p>
    <form @submit.prevent="doCalc">
      <div class="calc-row">
        <label for="variant">Variant poistenia: </label>
        <div class="select is-primary">
          <select id="variant" v-model="variantIns">
            <option value="short">krátkodobé poistenie</option>
            <option value="long">celoročné poistenie</option>
          </select>
        </div>
      </div>
      <div class="calc-row">
        <label for="start">Začiatok poistenia: </label>
        <input
          id="start"
          class="date-style"
          :class="{ error: startDateEmpty }"
          type="date"
          v-model="startOfIns"
        />
      </div>
      <div class="calc-row" v-if="!checkFull">
        <label for="end">Koniec poistenia: </label>
        <input
          id="end"
          class="date-style"
          :class="{ error: endDateEmpty }"
          type="date"
          v-model="endOfIns"
        />
      </div>
      <div class="calc-row">
        <label>Balík:</label>
        <div class="control">
          <label class="radio">
            <input
              type="radio"
              name="balik"
              value="0"
              v-model="packetIns"
              checked
            />
            základný</label
          >
          <label class="radio">
            <input type="radio" name="balik" value="1" v-model="packetIns" />
            rozšírený</label
          >
          <label class="radio">
            <input type="radio" name="balik" value="2" v-model="packetIns" />
            extra</label
          >
        </div>
      </div>

      <div class="calc-row">
        <label>Pripoistenia: </label>
        <div>
          <label class="checkbox">
            <input type="checkbox" value="true" v-model="stornoIns" />
            Storno cesty
          </label>
          <label class="checkbox">
            <input type="checkbox" value="true" v-model="sportIns" />
            športové aktivity
          </label>
        </div>
      </div>

      <div class="calc-row">
        <label for="persons">Počet osôb: </label>
        <div class="select is-primary">
          <select id="persons" required v-model="personsIns">
            <option value="1">1</option>
            <option value="2">2</option>
            <option value="3">3</option>
          </select>
        </div>
      </div>
      <br />
      <button type="submit" class="button is-primary">Vypočítaj</button>
    </form>

    <div>
      <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
      <p v-else>
        Výška Vášho poistného je: <strong>{{ result }}</strong>
      </p>
    </div>
  </div>
</template>

<style scoped lang="scss">
@import 'bulma/sass/utilities/_all.sass';
@import 'bulma/sass/form/_all.sass';
@import 'bulma/sass/elements/_all.sass';

h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

.hello {
  width: 30%;
  margin: 0 auto;
}

.calc-row {
  display: flex;
  justify-content: space-between;
  margin: 1em;
  padding-bottom: 0.5em;
  border-bottom: 1px solid #ddd;
  label {
    margin-top: 0.75em;
  }
}

.date-style {
  border: 1px solid;
  border-color: #00b89c;
  padding: 1em;
  border-radius: 4px;
}

.error {
  border-color: red;
}

.error-message {
  color: red;
}
</style>
