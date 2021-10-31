<template>
  <div>
    <table
      v-if="loaded"
      class="styled-table"
      style="margin-left: auto; margin-right: auto;"
    >
      <thead>
        <tr>
          <th class="widthControlledCell"></th>
          <th
            v-for="(date, index) in daysList"
            :key="index"
            class="widthControlledCell"
          >
            {{ date }}
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(habit, index) in habits" :key="index + 'A' + reloads.table">
          <!-- Habit name -->
          <td @click="showHabitDetails(index)" style="cursor: pointer;">
            {{ habit.name }}
          </td>
          <!-- Before started -->
          <td
            v-for="index2 in Math.min(
              Math.max(
                new Date(habit.startDay) - new Date(firstDisplayedDate),
                0
              ) / 86400000,
              settings.displayedDays
            )"
            :key="'B' + index2"
            class="widthControlledCell"
          ></td>
          <!-- After started new -->
          <td
            class="widthControlledCell"
            v-for="(date, index) in fullDaysList(
              new Date(habit.startDay) > new Date(firstDisplayedDate)
                ? new Date(habit.startDay)
                : new Date(firstDisplayedDate)
            )"
            :key="'fd' + index"
          >
            <div
              v-if="new Date(date) <= new Date()"
              style="cursor: pointer;"
              @click="changeStatus(habit, date)"
            >
              <!-- QUESTION MARK, SLIM YET TO BE FOUND -->
              <v-icon v-if="!(date in habit.records)" color="#444444"
                >mdi-square-rounded-outline</v-icon
              >
              <!-- help / checkbox-blank-outline / square-outline / square-rounded-outline ... might be better? -->
              <!-- TICK -->
              <v-icon
                v-if="date in habit.records && habit.records[date]"
                color="green darken-2"
                >{{
                  settings.useThick ? "mdi-check-bold" : "mdi-check"
                }}</v-icon
              >
              <!-- CROSS -->
              <v-icon
                v-if="date in habit.records && !habit.records[date]"
                color="red darken-2"
                >{{
                  settings.useThick ? "mdi-close-thick" : "mdi-window-close"
                }}</v-icon
              >
            </div>
            <div v-if="new Date(date) > new Date()">
              <!-- Future -->
              <!-- <v-icon v-if="!(date in habit.records)" color="#444444"
                >mdi-minus</v-icon
              > -->
            </div>
          </td>
          <!-- After started old -->
          <!-- <td v-for="(cell, index2) in habit.dayRecords" :key="'C' + index2">
            {{ cell }}
          </td>
          <td
            v-for="index2 in Math.ceil(
              (currentDate -
                habit.startDay -
                habit.dayRecords.length * 86400000) /
                86400000
            )"
            :key="'D' + index2"
          >
            -
          </td> -->
        </tr>
      </tbody>
    </table>
    <div v-if="this.loaded" id="settingsDiv">
      <!-- thick icons -->
      <v-switch
        v-model="settings.useThick"
        label="Use thick icons"
        @change="updateSettings"
      ></v-switch>
      <!-- days ago -->
      <v-text-field
        label="Days ago"
        persistent-hint
        v-model="settings.daysAgo"
        :hint="
          'last displayed day will be set to ' + settings.daysAgo + ' days ago'
        "
        type="number"
        @change="updateSettings"
      ></v-text-field>
      <v-slider
        v-model="settings.daysAgo"
        :max="this.getStartedDaysAgo() - settings.displayedDays + 2"
        :min="0"
        step="1"
        persistent-hint
        :label="'Days ago: ' + settings.daysAgo"
        @change="updateSettings"
      ></v-slider>
      <!-- displayed days -->
      <v-slider
        v-model="settings.displayedDays"
        :max="10"
        :min="1"
        step="1"
        persistent-hint
        :label="'Displayed days: ' + settings.displayedDays"
        @change="updateSettings"
      ></v-slider>
      <v-container>
        <v-row>
          <v-text-field
            id="newHabitNameInput"
            label="Add a new habit"
            persistent-hint
          ></v-text-field>
          <v-btn
            @click="addNewHabit"
            style="margin-top: 16px; margin-left: 16px;"
            >add habit</v-btn
          >
        </v-row>
      </v-container>
    </div>
    <!-- HABIT DETAILS V-DIALOG -->
    <v-dialog v-model="habitDetailsDialog" width="min(80%, 600px)">
      <v-card v-if="loaded">
        <v-card-title class="text-h5 grey lighten-2">
          {{
            this.habitDetailsDialog ? this.habits[displayedHabitId].name : ""
          }}
          <v-icon
            v-if="this.habits.length > 1"
            color="red"
            style="position: absolute; right: 20px; cursor: pointer;"
            @click="deleteHabit"
            >mdi-delete</v-icon
          >
        </v-card-title>

        <v-card-text style="margin-top: 16px;">
          dzień rozpoczęcia:
          {{
            this.habitDetailsDialog
              ? dateToYYYYMMDD(this.habits[displayedHabitId].startDay)
              : ""
          }}
          <br />
          dni z rzędu: {{ getDaysInARow(displayedHabitId) }}
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  data: () => ({
    habitDetailsDialog: false,
    loaded: false,
    habits: [],
    displayedHabitId: 0,
    currentDate: new Date(),
    settings: {},
    reloads: {
      table: 1,
    },
  }),
  methods: {
    getStartedDaysAgo: function() {
      let firstStartDay = Math.floor(
        (new Date() - this.getFirstStartDay()) / 86400000
      );
      console.warn(firstStartDay);
      return firstStartDay;
    },
    getFirstStartDay: function() {
      let earliestStartDay = new Date();
      for (let habit of this.habits) {
        let checkedDate = new Date(habit.startDay);
        console.error(checkedDate);
        console.warn(checkedDate > earliestStartDay);
        console.warn(checkedDate == earliestStartDay);
        console.warn(checkedDate < earliestStartDay);
        if (checkedDate < earliestStartDay) {
          earliestStartDay = checkedDate;
          console.warn(earliestStartDay);
        }
      }
      console.error("earliestStartDay");
      console.error(earliestStartDay);
      return earliestStartDay;
    },
    deleteHabit: function() {
      this.habits.splice(this.displayedHabitId, 1);
      this.habitDetailsDialog = false;
      this.refreshAndUpdate();
    },
    dateToYYYYMMDD: function(date) {
      return new Date(date).toISOString().substring(0, 10);
    },
    showHabitDetails: function(habitIndex) {
      this.displayedHabitId = habitIndex;
      this.habitDetailsDialog = true;
    },
    getDaysInARow: function(habitIndex) {
      let habit = this.habits[habitIndex];
      let daysInARow = 0;
      console.log(habit + daysInARow);
      while (
        habit.records[
          new Date(new Date() - 86400000 * daysInARow)
            .toISOString()
            .substring(0, 10)
        ]
      ) {
        daysInARow += 1;
      }
      return daysInARow;
    },
    addNewHabit: function() {
      this.habits.push({
        name: document.getElementById("newHabitNameInput").value,
        startDay: new Date(new Date().toISOString().substring(0, 10)),
        records: {},
      });
      this.refreshAndUpdate();
    },
    updateSettings: function() {
      localStorage.setItem("settings", JSON.stringify(this.settings));
    },
    refreshAndUpdate: function() {
      this.reloads.table += 1;
      localStorage.setItem("habits", JSON.stringify(this.habits));
    },
    changeStatus: function(habit, date) {
      if (habit.records[date]) {
        habit.records[date] = false;
      } else {
        habit.records[date] = true;
      }
      this.refreshAndUpdate();
    },
    loadData: function() {
      let storedSettings = localStorage.getItem("settings");
      if (storedSettings) {
        this.settings = JSON.parse(storedSettings);
      } else {
        this.settings.useThick = false;
        this.settings.displayedDays = 8;
        this.settings.daysAgo = 0;
      }
      let storedHabits = localStorage.getItem("habits");
      if (storedHabits) {
        this.habits = JSON.parse(storedHabits);
        console.log("from localData:");
        console.log(this.habits);
        console.log("length from localData:");
        console.log(this.habits.length);
        console.log("stringified from localData: ");
        console.log(JSON.stringify(this.habits));
      } else {
        console.log("no data found in localstorage!");
        this.habits.push(
          {
            name: "powygrzewać się na słoneczku",
            startDay: new Date("2021-10-12"),
            dayRecords: [5, 6, 8],
            records: {
              "2021-10-12": true,
              "2021-10-15": true,
              "2021-10-17": true,
              "2021-10-18": false,
            },
          },
          {
            name: "wejść w pudełko",
            startDay: new Date("2021-10-13"),
            dayRecords: [3, 4, 6],
            records: {
              "2021-10-13": true,
              "2021-10-14": true,
              "2021-10-15": true,
            },
          },
          {
            name: "ponosić biszkopta w pyszczku",
            startDay: new Date("2021-10-13"),
            dayRecords: [2, 3],
            records: {
              "2021-10-13": true,
              "2021-10-14": false,
              "2021-10-15": true,
              "2021-10-16": true,
            },
          },
          {
            name: "zeżreć puchę karmy",
            startDay: new Date("2021-10-13"),
            dayRecords: [1, 1],
            records: {
              "2021-10-13": true,
              "2021-10-14": true,
              "2021-10-15": true,
              "2021-10-16": true,
              "2021-10-17": true,
              "2021-10-18": true,
              "2021-10-19": true,
            },
          },
          {
            name: "być grzecznym",
            startDay: new Date("2021-10-13"),
            dayRecords: [1, 1],
            records: {
              "2021-10-13": true,
              "2021-10-14": true,
              "2021-10-15": false,
              "2021-10-16": false,
              "2021-10-17": false,
              "2021-10-18": false,
              "2021-10-19": false,
            },
          },
          {
            name: "wylizać sobie futerko",
            startDay: new Date("2021-10-12"),
            dayRecords: [1, 1],
            records: {
              "2021-10-12": true,
              "2021-10-13": true,
              "2021-10-14": false,
              "2021-10-15": true,
              "2021-10-16": true,
              "2021-10-17": false,
              "2021-10-18": true,
              "2021-10-19": true,
            },
          },
          {
            name: "wylizać komuś innemu futerko",
            startDay: new Date("2021-10-12"),
            dayRecords: [1, 1],
            records: {
              "2021-10-12": true,
              "2021-10-13": false,
              "2021-10-14": true,
              "2021-10-16": true,
              "2021-10-18": false,
              "2021-10-19": true,
            },
          }
        );
        console.log("pushed: ");
        console.log(this.habits);
        console.log("stringified pushed: ");
        console.log(JSON.stringify(this.habits));
      }
      this.loaded = true;
    },
    fullDaysList: function(sinceDate) {
      console.log("fullDaysList");
      let iDate = new Date(sinceDate);
      let dates = [];
      while (iDate <= this.lastDisplayedDate) {
        dates.push(
          iDate.getFullYear() +
            "-" +
            (iDate.getMonth() + 1) +
            "-" +
            iDate.getDate()
        );
        iDate.setDate(iDate.getDate() + 1);
      }
      return dates;
    },
  },
  mounted() {
    this.loadData();
    console.log(JSON.stringify(this.habits));
  },
  computed: {
    lastDisplayedDate: function() {
      let lastDate = new Date(
        new Date(new Date() - 86400000 * this.settings.daysAgo)
          .toISOString()
          .substring(0, 10)
      );
      console.log("lastDisplayedDate: ");
      console.log(lastDate);
      return lastDate;
    },
    firstDisplayedDate: function() {
      console.log("typeof daysAgo");
      console.log(typeof this.settings.daysAgo);
      let firstDate = new Date(
        new Date(
          new Date() -
            86400000 *
              (this.settings.displayedDays -
                1 +
                parseInt(this.settings.daysAgo))
        )
          .toISOString()
          .substring(0, 10)
      );
      console.log("firstDisplayedDate");
      console.log(firstDate);
      console.log("this.settings.displayedDays - 1 + this.settings.daysAgo");
      console.log(this.settings.displayedDays - 1 + this.settings.daysAgo);
      return firstDate;
    },
    daysList: function() {
      let iDate = new Date(this.firstDisplayedDate.valueOf());
      let dates = [];
      while (iDate <= this.lastDisplayedDate) {
        // display month as well
        //dates.push(iDate.getMonth() + 1 + "-" + iDate.getDate());
        // only display day
        dates.push(iDate.getDate());
        iDate.setDate(iDate.getDate() + 1);
      }
      // dates.pop();
      // dates.push("today");
      console.log("daysList");
      console.log(dates);
      return dates;
    },
  },
};
</script>

<style scoped>
#settingsDiv {
  max-width: 600px;
  margin-left: calc(50% - 300px);
  padding: 20px;
  border: 1px solid #22aadd;
  border-radius: 12px;
  background: #effbfd;
}

.widthControlledCell {
  text-align: center;
  min-width: 54px;
  height: 50px;
}

.styled-table {
  border-collapse: collapse;
  margin: 25px 0;
  font-size: 1em;
  font-family: sans-serif;
  min-width: 250px;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.15);
}

.styled-table thead tr {
  background-color: #22aadd;
  color: #ffffff;
  text-align: left;
}

.styled-table th,
.styled-table td {
  padding: 12px 15px;
}

.styled-table tbody tr {
  border-bottom: 1px solid #dddddd;
}

.styled-table tbody tr:nth-of-type(even) {
  background-color: #f3f3f3;
}

.styled-table tbody tr:last-of-type {
  border-bottom: 2px solid #22aadd;
}

.styled-table tbody tr.active-row {
  font-weight: bold;
  color: #22aadd;
  /* #009879; */
}
</style>
