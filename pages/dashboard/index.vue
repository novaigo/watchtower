<template>
  <section>
    <b-table
      class="game-table"
      :data="isEmpty ? [] : data"
      :bordered="isBordered"
      :striped="isStriped"
      :narrowed="isNarrowed"
      :hoverable="isHoverable"
      :loading="isLoading"
      :focusable="isFocusable"
      :mobile-cards="hasMobileCards"
    >
      <b-table-column field="id" label="ID" width="10" numeric v-slot="props">
        {{ props.row.id }}
      </b-table-column>

      <b-table-column field="location" label="Location" v-slot="props">
        {{ props.row.location }}
      </b-table-column>

      <b-table-column field="description" label="Description" v-slot="props">
        {{ props.row.description }}
      </b-table-column>

      <b-table-column field="status" label="Status" centered v-slot="props">
        <b-switch
          v-model="props.row.status"
          :disabled="isGameOver"
          :type="props.row.status ? 'is-success' : 'is-danger'"
        ></b-switch>
      </b-table-column>

      <b-table-column label="Battery" v-slot="props">
        <span :style="{ color: props.row.battery < 10 ? 'red' : 'inherit' }">
          {{ props.row.battery + `%` }}</span
        >
      </b-table-column>
    </b-table>

    <div class="has-text-centered">
      <b-button
        type="is-primary"
        @click="startNewGame"
        :disabled="isStartingNewGame"
      >
        Start New Game
      </b-button>
      <p class="game-description is-size-6 has-text-centered">
        The objective of the watchtower game is to maintain home security by
        monitoring device status and battery levels. Players must strategically
        toggle device status between active and disabled to conserve battery
        while facing the threat of a thief attempting to infiltrate the house
        every 10 seconds. Quick reactions are key to preventing security
        breaches. Be cautious, as batteries below 10% take twice as long to
        charge!
      </p>
    </div>
  </section>
</template>

<script>
export default {
  data() {
    const initialBatteryValue = Array.from({ length: 5 }, () =>
      Math.floor(Math.random() * 101)
    );

    const data = [
      {
        id: 1,
        location: "Front door",
        description: "Doorbell Camera",
        status: true,
        battery: initialBatteryValue[0],
      },
      {
        id: 2,
        location: "Front door",
        description: "Alarm",
        status: true,
        battery: initialBatteryValue[1],
      },
      {
        id: 3,
        location: "Patio",
        description: "Smart lock",
        status: true,
        battery: initialBatteryValue[2],
      },
      {
        id: 4,
        location: "Garage",
        description: "Carbon monoxide detector",
        status: true,
        battery: initialBatteryValue[3],
      },
      {
        id: 5,
        location: "Garage",
        description: "Motion sensor",
        status: true,
        battery: initialBatteryValue[4],
      },
    ];

    return {
      data,
      isEmpty: false,
      isBordered: true,
      isStriped: true,
      isNarrowed: false,
      isHoverable: false,
      isFocusable: false,
      isLoading: false,
      hasMobileCards: true,
      batteryInterval: null,
      initialBatteryValue,
      isStartingNewGame: false,
    };
  },
  computed: {
    isGameOver() {
      return this.data.every((row) => row.battery === 0);
    },
  },
  methods: {
    batteryTimer() {
      this.batteryInterval = setInterval(() => {
        this.data.forEach((row) => {
          if (row.status && row.battery > 0) {
            row.battery -= 1;
          } else if (row.status && row.battery === 0) {
            row.status = false;
          } else if (!row.status && row.battery < 10) {
            row.battery += 0.5;
          } else if (!row.status && row.battery >= 10 && row.battery < 100) {
            row.battery += 2;
          }
        });
      }, 1000);
    },
    thiefTimer() {
      const timer = setInterval(() => {
        const locations = this.data.map((row) => row.location);
        const randomLocation =
          locations[Math.floor(Math.random() * locations.length)];
        const targetedRow = this.data.find(
          (row) => row.location === randomLocation
        );
        if (targetedRow) {
          if (!targetedRow.status) {
            clearInterval(timer);
            clearInterval(this.batteryInterval);
            this.data.forEach((row) => {
              row.battery = 0;
            });
            this.data.forEach((row) => {
              row.batteryColor = "red";
            });
          }
        }
      }, 10000);
    },
    startNewGame() {
      this.isStartingNewGame = true;
      setTimeout(() => {
        this.isStartingNewGame = false;
      }, 10000);

      this.data.forEach((row, i) => {
        row.status = true;
        row.battery = this.initialBatteryValue[i];
      });
      this.batteryTimer();
      this.thiefTimer();
    },
  },
};
</script>

<style scoped>
.game-description {
  padding-top: 2em;
  max-width: 50%;
  margin: 0 auto;
}

.game-table {
  padding: 1em 3em;
  margin: 0 auto;
}
</style>
