<template>
  <div v-for="(payer, payerId) in fillPayersArray" :key="payer">
    <v-table v-if="checkZeroDebtors(payer)" density="compact" class="border rounded-lg pa-2 mb-10">
      <thead>
        <tr>
          <th colspan="2">
            <strong class="text-uppercase text-h6">
              {{ storePeople.findPerson(payerId).name }} должны
            </strong>
          </th>
        </tr>
        <tr>
          <th class="center">
            <strong class="text-uppercase">имя</strong>
          </th>
          <th class="center">
            <strong class="text-uppercase">сумма долга</strong>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(debtor, debtorId) in payer.debtors" :key="debtorId">
          <th v-if="debtor != 0" class="center">
            {{ storePeople.findPerson(debtorId).name }}
          </th>
          <th v-if="debtor != 0" class="center">
            {{ debtor }}
          </th>
        </tr>
      </tbody>
    </v-table>
  </div>
</template>

<script>
import { useFoodsStore } from '@/stores/foods'
import { usePeopleStore } from '@/stores/people'

export default {
  data() {
    return {
      storeFoods: useFoodsStore(),
      storePeople: usePeopleStore(),
      payers: {}
    }
  },
  computed: {
    // Заполняет массив платильщиков-должников
    fillPayersArray: function () {
      let payersArray = []
      this.storeFoods.foods.forEach((food) => {
        if (!(food.payerId in payersArray)) {
          payersArray.push(food.payerId)
        }
      })

      // заполнение payers дефолтными значениями
      payersArray.forEach((payer) => {
        this.payers[payer] = {
          debtors: {}
        }
      })

      // заполнение списка должников для каждого платильщика
      this.storeFoods.foods.forEach((food) => {
        food.eaters.forEach((eater) => {
          if (eater.id != food.payerId) {
            if (!(eater.id in this.payers[food.payerId].debtors)) {
              this.payers[food.payerId].debtors[eater.id] = Math.floor(
                food.cost / food.eaters.length
              )
            } else {
              this.payers[food.payerId].debtors[eater.id] += Math.floor(
                food.cost / food.eaters.length
              )
            }
          }
        })
      })

      // перерасчет сумм должников, для случаев раздельного счета
      for (let debtorCurrentPerson in this.payers) {
        for (let debtorComparablePerson in this.payers) {
          if (debtorCurrentPerson != debtorComparablePerson) {
            if (debtorCurrentPerson in this.payers[debtorComparablePerson].debtors && debtorComparablePerson in this.payers[debtorCurrentPerson].debtors) {
              if (this.payers[debtorComparablePerson].debtors[debtorCurrentPerson] == this.payers[debtorCurrentPerson].debtors[debtorComparablePerson]) {
                this.payers[debtorComparablePerson].debtors[debtorCurrentPerson] = 0
                this.payers[debtorCurrentPerson].debtors[debtorComparablePerson] = 0
              } else if (this.payers[debtorComparablePerson].debtors[debtorCurrentPerson] > this.payers[debtorCurrentPerson].debtors[debtorComparablePerson]) {
                this.payers[debtorComparablePerson].debtors[debtorCurrentPerson] -= this.payers[debtorCurrentPerson].debtors[debtorComparablePerson]
                this.payers[debtorCurrentPerson].debtors[debtorComparablePerson] = 0
              } else {
                this.payers[debtorCurrentPerson].debtors[debtorComparablePerson] -= this.payers[debtorComparablePerson].debtors[debtorCurrentPerson]
                this.payers[debtorComparablePerson].debtors[debtorCurrentPerson] = 0
              }
            }
          }
        }
      }

      return this.payers
    }
  },
  methods: {
    // Проверка на наличие должников с нудевой суммой
    checkZeroDebtors(payer) {
      let hasDebtor = false

      for (let debtor in payer.debtors) {
        if (payer.debtors[debtor] != 0) {
          hasDebtor = true
        }
      }

      return hasDebtor
    }
  }
}
</script>
