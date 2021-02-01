<template>
  <q-page class="row items-center justify-evenly">
    <q-card class="my-card bg-grey-2 text-black">
      <q-card-section>
        <div class="text-h6 text-center">Zgadnij liczbę</div>
        <div class="text-subtitle2 text-center">{{ "Wprowadź liczbe od "+minValue+" do "+maxValue+", a ja powiem czy zgadłeś" }}</div>
      </q-card-section>

      <q-card-section>
        <q-input filled v-model="valueInBox" label="Liczba" type="number" />
      </q-card-section>

      <q-card-section>
        <div class="text-subtitle2 text-center">{{ infoString }}</div>
      </q-card-section>

      <q-card-actions>
        <q-btn class="fit" color="positive" @click="guess">Zgadnij</q-btn>
      </q-card-actions>
      <q-card-actions>
        <q-btn class="fit" @click="settingsDialog = true">Zmień ustawienia</q-btn>
      </q-card-actions>
    </q-card>

    <q-card class="my-card bg-grey-2 text-black" v-if="leaderBoardData.length > 0">
      <q-table
        title="Tablica rankingów"
        :data="leaderBoardData"
        :columns="leaderBoardColumns"
        row-key="id"
      />
    </q-card>

    <q-dialog v-model="settingsDialog">
      <q-card>
        <q-card-section>
          <div class="text-h6">Ustawienia</div>
        </q-card-section>

        <q-card-section>
          <q-input filled v-model="minValue" label="Liczba minimalna" type="number" />
        </q-card-section>
        <q-card-section>
          <q-input filled v-model="maxValue" label="Liczba maksymalna" type="number" />
        </q-card-section>

        <q-card-actions>
          <q-btn flat label="OK" color="primary" v-close-popup class="fit"/>
        </q-card-actions>
      </q-card>
    </q-dialog>

    <q-dialog v-model="winDialog" persistent>
      <q-card>
        <q-card-section>
          <div class="text-h6">Wooohoo! Zgadłeś</div>
          <div class="text-subtitle2 text-center">{{ infoString }}</div>
        </q-card-section>

        <q-card-section>
          <q-input filled v-model="winDialogName" label="Jak masz na imie?" />
        </q-card-section>

        <q-card-actions>
          <q-btn flat label="OK" color="primary" v-close-popup class="fit"/>
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script lang="ts">
import { Vue, Component, Watch } from 'vue-property-decorator'

const leaderBoardColumns = [
  {
    name: 'name',
    label: 'Imie',
    field: 'name',
    sortable: true
  },
  {
    name: 'range',
    label: 'Ile liczb (trudność)',
    field: 'range',
    sortable: true
  },
  {
    name: 'tryCount',
    label: 'Sprób',
    field: 'tryCount',
    sortable: true
  }
]

interface LeaderBoardEntry {
  id: number,
  name: string,
  range: number,
  tryCount: number
}

@Component({})
export default class PageIndex extends Vue {
  actualValue = 0
  lastValue = null as null | number
  valueInBox = null as null | string
  tryCount = 0

  settingsDialog = false
  minValue = 0
  maxValue = 100

  winDialog = false
  winDialogName = null

  leaderBoardData = [] as LeaderBoardEntry[]
  leaderBoardColumns = leaderBoardColumns

  get infoString (): string {
    if (this.lastValue !== null) {
      if (this.lastValue === this.actualValue) {
        return `Zgadłeś za ${this.tryCount} razem. Sprobujesz jeszcze raz?`
      }
      if (this.lastValue > this.actualValue) {
        return 'Ehh... nie udało się. Za dużo.'
      }
      return 'Ehh... nie udało się. Za mało.'
    }
    return ''
  }

  get winCondition (): boolean {
    return this.lastValue === this.actualValue
  }

  @Watch('winCondition')
  winConditionChanged () {
    this.winDialog = this.winCondition
  }

  @Watch('winDialog')
  winDialogChanged () {
    if (!this.winDialog) {
      if (this.winDialogName !== null && this.winDialogName !== '') { this.addToLeaderBoard(<string><unknown> this.winDialogName) }
      this.newGame()
      this.valueInBox = null
    }
  }

  @Watch('settingsDialog')
  settingsDialogChanged () {
    this.newGame()
  }

  addToLeaderBoard (name: string) {
    this.leaderBoardData.push({
      id: this.leaderBoardData.length,
      name,
      range: this.maxValue - this.minValue,
      tryCount: this.tryCount
    })
  }

  guess () {
    if (this.winCondition) { this.newGame() }

    if (this.valueInBox !== null) {
      this.tryCount += 1
      this.lastValue = parseInt(this.valueInBox)
    }
  }

  newGame () {
    this.tryCount = 0
    this.lastValue = null
    this.actualValue = Math.floor(Math.random() * (this.maxValue - this.minValue + 1)) + this.minValue
  }

  mounted () {
    this.newGame()
  }
}
</script>
