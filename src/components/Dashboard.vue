<template>
  <div>
    <table class="styled-table">
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
        <tr v-for="(habit, index) in stuff" :key="'A' + index">
          <!-- Habit name -->
          <td>{{ habit.name }}</td>
          <!-- Before started -->
          <td
            v-for="index2 in Math.min(
              Math.max(habit.startDay - firstDisplayedDate, 0) / 86400000,
              settings.displayedDays
            )"
            :key="'B' + index2"
            class="widthControlledCell"
          ></td>
          <!-- After started new -->
          <td
            class="widthControlledCell"
            v-for="(date, index) in fullDaysList(
              habit.startDay > firstDisplayedDate
                ? habit.startDay
                : firstDisplayedDate
            )"
            :key="'fd' + index"
          >
            <div v-if="new Date(date) <= new Date()" style="cursor: pointer;">
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
    <v-col cols="12" sm="6">
      <v-switch v-model="settings.useThick" label="Use thick icons"></v-switch>
      <v-text-field
        label="Days ago"
        persistent-hint
        v-model="settings.daysAgo"
        outlined
        :hint="
          'last displayed day will be set to ' + settings.daysAgo + ' days ago'
        "
        type="number"
      ></v-text-field>
      <v-slider
        v-model="settings.displayedDays"
        :max="10"
        :min="1"
        step="1"
        persistent-hint
        :label="'Displayed days: ' + settings.displayedDays"
      ></v-slider>
    </v-col>
  </div>
</template>

<script>
export default {
  data: () => ({
    stuff: [],
    currentDate: new Date(),
    settings: {
      useThick: false,
      displayedDays: 8,
      daysAgo: 0,
    },
  }),
  methods: {
    loadData: function() {
      this.stuff.push(
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
            "2021-10-13": false,
            "2021-10-14": false,
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
    },
    fullDaysList: function(sinceDate) {
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
      return dates;
    },
  },
};
</script>

<style scoped>
.widthControlledCell {
  text-align: center;
  min-width: 54px;
  min-height: 50px;
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
