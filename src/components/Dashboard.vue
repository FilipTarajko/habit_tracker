<template>
  <div id="content">
    <table
      v-if="loaded"
      class="styled-table"
      style="margin-left: auto; margin-right: auto;"
    >
      <thead>
        <tr>
          <th class="widthControlledCell"></th>
          <th
            v-for="(date, index) in fullDaysList()"
            :key="index"
            class="widthControlledCell"
          >
            {{ date.substr(5, 6) }}
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
                new Date(habit.startDay) - new Date(fullDaysList()[0]),
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
              new Date(habit.startDay) > new Date(fullDaysList()[0])
                ? new Date(habit.startDay) - 86400000
                : undefined,
              index
            )"
            :key="'fd' + index"
          >
            <div
              style="cursor: pointer;"
              @click="changeStatus(habit, date)"
              :class="
                date == (habit.startDay + '').substring(0, 10) &&
                settings.markStartDay
                  ? 'firstDayOfHabitTableDiv'
                  : ''
              "
            >
              <span v-if="settings.showCellDate" style="font-size:12px;">{{
                date.substr(5, 5)
              }}</span>
              <!-- QUESTION MARK, SLIM YET TO BE FOUND -->
              <v-icon v-if="!(date in habit.records)" color="#444444"
                >mdi-square-rounded-outline</v-icon
              >
              <!-- help / checkbox-blank-outline / square-outline / square-rounded-outline ... might be better? -->
              <!-- TICK -->
              <v-icon
                v-if="date in habit.records && habit.records[date]"
                :color="settings.blueGreen ? 'blue darken-1' : 'green darken-2'"
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
      <!-- accent color -->
      <v-color-picker
        v-model="settings.color"
        dot-size="25"
        swatches-max-height="200"
        @input="updateColor"
      ></v-color-picker>

      <!-- mark start day -->
      <v-switch
        v-model="settings.markStartDay"
        label="Mark start day"
        @change="updateSettings"
      ></v-switch>
      <!-- show cell date -->
      <v-switch
        v-model="settings.showCellDate"
        label="Show cell date"
        @change="updateSettings"
      ></v-switch>
      <!-- deuteranopia colorblind mode -->
      <v-switch
        v-model="settings.blueGreen"
        label="Colorblind mode"
        @change="updateSettings"
      ></v-switch>
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
    rgbToHsl: function(color) {
      var r = parseInt((color + "").substr(1, 2), 16); // Grab the hex representation of red (chars 1-2) and convert to decimal (base 10).
      var g = parseInt((color + "").substr(3, 2), 16);
      var b = parseInt((color + "").substr(5, 2), 16);

      (r /= 255), (g /= 255), (b /= 255);
      var max = Math.max(r, g, b),
        min = Math.min(r, g, b);
      var h,
        s,
        l = (max + min) / 2;

      if (max == min) {
        h = s = 0; // achromatic
      } else {
        var d = max - min;
        s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
        switch (max) {
          case r:
            h = (g - b) / d + (g < b ? 6 : 0);
            break;
          case g:
            h = (b - r) / d + 2;
            break;
          case b:
            h = (r - g) / d + 4;
            break;
        }
        h /= 6;
      }

      return [h, s, l];
    },
    updateColor: function() {
      // console.log(this.rgbToHsl(this.settings.color)[0] * 360);
      // console.log(this.rgbToHsl(this.settings.color)[2]);
      let content = document.querySelector("#content");
      content.style.setProperty(
        "--hue",
        this.rgbToHsl(this.settings.color)[0] * 360
      );
      content.style.setProperty(
        "--value",
        "" + this.rgbToHsl(this.settings.color)[2] * 100 + "%"
      );
      this.updateSettings();
    },
    getStartedDaysAgo: function() {
      let firstStartDay = Math.floor(
        (new Date() - this.getFirstStartDay()) / 86400000
      );
      //console.log(firstStartDay);
      return firstStartDay;
    },
    getFirstStartDay: function() {
      let earliestStartDay = new Date();
      for (let habit of this.habits) {
        let checkedDate = new Date(habit.startDay);
        //console.log(checkedDate);
        // console.log(checkedDate > earliestStartDay);
        // console.log(checkedDate == earliestStartDay);
        // console.log(checkedDate < earliestStartDay);
        if (checkedDate < earliestStartDay) {
          earliestStartDay = checkedDate;
          //console.log(earliestStartDay);
        }
      }
      //console.log("earliestStartDay");
      //console.log(earliestStartDay);
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
      //console.log(habit + daysInARow);
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
        this.settings.markStartDay = false;
        this.settings.showCellDate = false;
      }
      let storedHabits = localStorage.getItem("habits");
      if (storedHabits) {
        console.log("found data in localStorage!");
        this.habits = JSON.parse(storedHabits);
        // console.log("from localData:");
        // console.log(this.habits);
        // console.log("length from localData:");
        // console.log(this.habits.length);
        // console.log("stringified from localData: ");
        // console.log(JSON.stringify(this.habits));
        this.updateColor();
      } else {
        console.warn("no data found in localstorage!");
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
          }
        );
        // console.log("pushed: ");
        // console.log(this.habits);
        // console.log("stringified pushed: ");
        // console.log(JSON.stringify(this.habits));
      }
      this.loaded = true;
    },
    // eslint-disable-next-line no-unused-vars
    fullDaysList: function(sinceDate, msg = "") {
      // console.log("fullDaysList");
      if (!sinceDate) {
        sinceDate = new Date(0);
      }
      let iDate = new Date(new Date() - 86400000 * this.settings.daysAgo);
      let dates = [];
      while (
        dates.length < this.settings.displayedDays &&
        new Date(sinceDate) <= new Date(iDate)
      ) {
        dates.unshift(
          iDate.getFullYear() +
            "-" +
            (iDate.getMonth() + 1) +
            "-" +
            (iDate.getDate() > 10 ? iDate.getDate() : "0" + iDate.getDate())
        );
        iDate.setDate(iDate.getDate() - 1);
      }
      // console.log(msg);
      // console.log(dates);
      return dates;
    },
  },
  mounted() {
    this.loadData();
    //console.log(JSON.stringify(this.habits));
  },
};
</script>

<style scoped>
#content {
  --hue: 196;
  --saturation: 73%;
  --value: 50%;
  --value-light: 92%;
  --accent-color: hsl(
    var(--hue),
    var(--saturation),
    var(--value)
  ); /* #22aadd */
  --accent-color-light: hsl(
    var(--hue),
    var(--saturation),
    var(--value-light)
  ); /* #effbfd */
  --odd-table-row-background-color: #ffffff;
  --even-table-row-background-color: #f3f3f3;
  --line-between-table-rows-color: #dddddd;
  --table-header-text-color: #ffffff;
}

#settingsDiv {
  max-width: 600px;
  margin-left: calc(50% - 300px);
  padding: 20px;
  border: 1px solid var(--accent-color);
  border-radius: 12px;
  background: var(--accent-color-light);
}

.widthControlledCell {
  text-align: center;
  min-width: 72px;
  height: 50px;
}

.styled-table {
  border-collapse: collapse;
  margin: 25px 0;
  font-size: 1em;
  font-family: sans-serif;
  min-width: 250px;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.15);
  background: var(--odd-table-row-background-color);
}

.styled-table thead tr {
  background-color: var(--accent-color);
  color: var(--table-header-text-color);
  text-align: left;
}

.styled-table th,
.styled-table td {
  padding: 12px 15px;
}

.styled-table tbody tr {
  border-bottom: 1px solid var(--line-between-table-rows-color);
}

.styled-table tbody tr:nth-of-type(even) {
  background-color: var(--even-table-row-background-color);
}

.styled-table tbody tr:last-of-type {
  border-bottom: 2px solid var(--accent-color);
}

.styled-table tbody tr.active-row {
  font-weight: bold;
  color: var(--accent-color);
  /* #009879; */
}

.firstDayOfHabitTableDiv {
  background: hsl(100, 50%, 50%, 45%);
}
</style>
