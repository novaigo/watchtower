<template>
  <section>
    <!-- buefy props after :data -->
    <b-table
      class="game-table"
      :data="data"
      :bordered="isBordered"
      :striped="isStriped"
      :mobile-cards="hasMobileCards"
    >
      <b-table-column
        field="id"
        label="ID"
        max-width="10"
        centered
        numeric
        v-slot="props"
      >
        {{ props.row.id }}
      </b-table-column>

      <b-table-column
        field="location"
        label="Location"
        max-width="300"
        v-slot="props"
      >
        {{ props.row.location }}
      </b-table-column>

      <b-table-column
        field="description"
        label="Description"
        max-width="300"
        v-slot="props"
      >
        {{ props.row.description }}
      </b-table-column>

      <b-table-column
        field="status"
        label="Status"
        centered
        max-width="150"
        v-slot="props"
      >
        <b-switch
          v-model="props.row.status"
          :disabled="isGameOver"
          :type="props.row.status ? 'is-success' : 'is-danger'"
        ></b-switch>
      </b-table-column>

      <b-table-column label="Battery" v-slot="props" centered max-width="100">
        <span :style="{ color: props.row.battery < 10 ? 'red' : 'inherit' }">
          {{ props.row.battery + `%` }}</span
        >
      </b-table-column>
    </b-table>

    <div class="has-text-centered">
      <b-field label="Choose how frequently the thief arrives (in seconds)">
        <b-slider
          :disabled="gameOver"
          class="custom-slider"
          tooltip-always
          v-model="thiefInterval"
          :min="2"
          :max="20"
        ></b-slider>
      </b-field>
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
        while facing the threat of a thief attempting to infiltrate the house.
        You can select the difficulty of the game by setting how often the thief
        arrives; less frequently is, of course, easier. Quick reactions are key
        to preventing security breaches. Be cautious, as batteries below 10%
        take twice as long to charge!
      </p>
    </div>
  </section>
</template>

<script>
export default {
  data() {
    const initialBatteryValue = [];
    for (let i = 0; i < 5; i++) {
      initialBatteryValue.push(Math.floor(Math.random() * 101));
    }
    //data here obviously shouldn't be hardcoded, but fetched from somewhere
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
      batteryInterval: null,
      initialBatteryValue,
      isStartingNewGame: false,
      thiefInterval: 11,
      gameOver: false,
      //buefy props bellow
      isBordered: true,
      isStriped: true,
      hasMobileCards: true,
    };
  },
  //computed acts like derived state in React (read-only)
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
          } else if (!row.status && row.battery >= 10 && row.battery < 99) {
            row.battery += 2;
          } else if (!row.status && row.battery >= 10 && row.battery === 99) {
            row.battery += 1;
          }
        });
      }, 1000);
    },
    thiefTimer() {
      const intervalId = setInterval(() => {
        const locations = this.data.map((row) => row.location);
        const randomLocation =
          locations[Math.floor(Math.random() * locations.length)];
        const targetedRow = this.data.find(
          (row) => row.location === randomLocation
        );
        if (targetedRow) {
          if (!targetedRow.status) {
            this.clearIntervals();
            this.data.forEach((row) => {
              row.battery = 0;
              row.batteryColor = "red";
              //still not sure on how to use Reactivity, revisit this
              // this.$set(this, "gameOver", true);
              this.gameOver = true;
              //show message
              this.$buefy.toast.open({
                message: "The thief successfully entered the house. You lost!",
                type: "is-danger",
                duration: 2000,
              });
            });
            clearInterval(intervalId);
            this.gameOver = false;
          }
        }
      }, this.thiefInterval * 1000);
    },
    startNewGame() {
      this.data.forEach((row, i) => {
        row.status = true;
        row.battery = this.initialBatteryValue[i];
      });
      this.gameOver = true;
      this.batteryTimer();
      this.thiefTimer();
      //show message
      this.$buefy.toast.open({
        message: "Game started!",
        type: "is-success",
        duration: 1500,
      });
      this.isStartingNewGame = false;
    },
    clearIntervals() {
      clearInterval(this.batteryInterval);
      clearInterval(this.thiefInterval);
    },
  },
};
</script>

<style scoped>
.game-description {
  padding-bottom: 2.75em;
}

.game-description,
.custom-slider {
  padding-top: 2.75em;
  max-width: 50%;
  margin: 0 auto;
}

.game-table {
  padding: 1em 3em;
  margin: 0 auto;
  max-width: 1200px;
}
.custom-slider {
  padding-bottom: 1em;
}
</style>
