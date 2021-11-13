<template>
  <div id="content" :class="settings.darkMode ? 'dark' : ''">
    <table
      v-if="loaded"
      class="styled-table"
      style="margin-left: auto; margin-right: auto;"
    >
      <thead>
        <tr>
          <th class="widthControlledCell" style="width: 320px;">Task</th>
          <th class="widthControlledCell">Deadline</th>
          <th class="widthControlledCell">Time left</th>
          <th class="widthControlledCell">Status</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(task, index) in filteredTasks"
          :key="index + 'B' + reloads.table"
          class="widthControlledCell"
        >
          <th>
            {{ task.name }}
          </th>
          <th>
            {{ task.deadline }}
          </th>
          <th>
            <span :style="getTaskColor(task)">{{
              timeLeft(task.deadline)
            }}</span>
          </th>
          <th
            v-if="task.status == 'todo'"
            style="cursor: pointer;"
            @click="changeTaskStatus(task)"
          >
            <v-icon color="#444444">mdi-square-rounded-outline</v-icon>
          </th>
          <th
            v-if="task.status == 'done'"
            style="cursor: pointer;"
            @click="changeTaskStatus(task)"
          >
            <v-icon
              :color="settings.blueGreen ? 'blue darken-1' : 'green darken-2'"
              >{{ settings.useThick ? "mdi-check-bold" : "mdi-check" }}</v-icon
            >
          </th>
        </tr>
      </tbody>
    </table>
    <table
      v-if="loaded"
      class="styled-table"
      style="margin-left: auto; margin-right: auto;"
    >
      <thead>
        <tr>
          <th class="widthControlledCell">Habit</th>
          <th
            v-for="(date, index) in fullDaysList()"
            :key="index"
            class="widthControlledCell"
          >
            {{ date.substr(5, 5) }}
          </th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(habit, index) in filteredHabits"
          :key="index + 'A' + reloads.table"
        >
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
                ? new Date(habit.startDay)
                : undefined,
              index
            )"
            :key="'fd' + index"
          >
            <span v-if="settings.showCellDate" style="font-size:12px;">{{
              date.substr(5, 5)
            }}</span>
            <span v-if="settings.displayWeekday" style="user-select: none;">
              <span
                v-if="settings.showCellWeekday || true"
                style="font-size:12px;"
                >{{ new Date(date).getDay() }}</span
              >
              <span
                v-if="doesHabitUseWeekdayFromDate(habit, date)"
                style="font-size:12px; color: green;"
                >+</span
              >
              <span
                v-if="!doesHabitUseWeekdayFromDate(habit, date)"
                style="font-size:12px; color: red;"
                >-</span
              >
            </span>
            <span v-if="doesHabitUseWeekdayFromDate(habit, date)">
              <div
                v-if="habit.type == 'boolean'"
                style="cursor: pointer;"
                @click="changeBooleanStatus(habit, date)"
                :class="
                  date == (habit.startDay + '').substring(0, 10) &&
                  settings.markStartDay
                    ? 'firstDayOfHabitTableDiv'
                    : ''
                "
              >
                <v-icon :color="getBooleanHabitStateColor(habit, date)">{{
                  getBooleanHabitStateIcon(habit, date)
                }}</v-icon>
              </div>

              <div
                v-if="habit.type == 'numeric'"
                style="cursor: pointer;"
                @click="changeNumericStatus(habit, date)"
                :class="
                  date == (habit.startDay + '').substring(0, 10) &&
                  settings.markStartDay
                    ? 'firstDayOfHabitTableDiv'
                    : ''
                "
              >
                <span :style="getNumericHabitStateStyle(habit, date)">
                  {{
                    habit.records[date]
                      ? date in habit.records && habit.records[date]
                      : 0
                  }}
                </span>
              </div>
            </span>
          </td>
        </tr>
      </tbody>
    </table>
    <div v-if="this.loaded" id="settingsDiv">
      <div><b>Adding</b></div>
      <!-- add a new task -->
      <v-container>
        <v-row>
          <v-text-field
            id="newTaskNameInput"
            label="Add a new task"
            persistent-hint
          ></v-text-field>
          <v-menu
            v-model="newTaskDatePickerMenu"
            :close-on-content-click="false"
            :nudge-right="40"
            transition="scale-transition"
            offset-y
            min-width="auto"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                style="margin-left: 10px;"
                v-model="newTaskDate"
                label="Task deadline"
                readonly
                v-bind="attrs"
                v-on="on"
              ></v-text-field>
            </template>
            <v-date-picker
              v-model="newTaskDate"
              @input="newTaskDatePickerMenu = false"
            ></v-date-picker>
          </v-menu>
          <v-btn
            @click="addNewTask"
            style="margin-top: 16px; margin-left: 16px;"
            >add task
          </v-btn>
        </v-row>
      </v-container>
      <!-- add a new boolean habit -->
      <v-container style="margin-top: -16px;">
        <v-row>
          <v-text-field
            id="newBooleanHabitNameInput"
            label="Add a new true/false habit"
            persistent-hint
          ></v-text-field>
          <v-text-field
            id="newBooleanHabitEveryNthTimeInput"
            label="every n days"
            persistent-hint
            style="margin-left: 10px; width: 0px;"
          ></v-text-field>
          <v-btn
            @click="addNewBooleanHabit"
            style="margin-top: 16px; margin-left: 16px;"
            >add habit</v-btn
          >
        </v-row>
      </v-container>
      <!-- add a new numeric habit -->
      <v-container style="margin-top: -16px;">
        <v-row>
          <v-text-field
            id="newNumericHabitNameInput"
            label="numeric habit name"
            persistent-hint
          ></v-text-field>
          <v-text-field
            style="margin-left: 10px; width: 40px;"
            id="newNumericHabitTargetInput"
            label="target"
            persistent-hint
          ></v-text-field>
          <v-text-field
            id="newNumericHabitEveryNthTimeInput"
            label="every n days"
            persistent-hint
            style="margin-left: 10px; width: 84px;"
          ></v-text-field>
          <v-btn
            @click="addNewNumericHabit"
            style="margin-top: 16px; margin-left: 16px;"
            >add habit</v-btn
          >
        </v-row>
        <br />
        days for new habit
        <v-row style="margin-top: 0px;">
          <span
            style="margin: 4px 8px 4px;"
            v-for="weekday in [
              'monday',
              'tuesday',
              'wednesday',
              'thursday',
              'friday',
              'saturday',
              'sunday',
            ]"
            :key="weekday"
          >
            <v-checkbox
              v-model="useWeekdays[weekday]"
              :label="weekday.substr(0, 3)"
            ></v-checkbox>
          </span>
        </v-row>
      </v-container>
      <b>Browsing</b><br /><br />
      <!-- days ago -->
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
          <!-- hide completed habits -->
          <v-switch
            class="settingsSwitch"
            v-model="settings.hideCompletedHabits"
            label="Hide completed habits"
            @change="updateSettings"
          ></v-switch>
          <!-- hide habits completed today -->
          <v-switch
            class="settingsSwitch"
            v-model="settings.hideHabitsCompletedToday"
            :disabled="settings.hideCompletedHabits"
            label="Hide habits completed today"
            @change="updateSettings"
          ></v-switch>
          <!-- hide completed tasks -->
          <v-switch
            class="settingsSwitch"
            v-model="settings.hideCompletedTasks"
            label="Hide completed tasks"
            @change="updateSettings"
          ></v-switch>
          <!-- sort tasks by deadline -->
          <v-switch
            class="settingsSwitch"
            v-model="settings.sortTasksByDeadline"
            label="Sort tasks by deadline"
            @change="updateSettings"
          ></v-switch>
        </v-row>
      </v-container>
      <b>Visual preferences</b>
      <br /><br />
      <v-container
        ><v-row>
          <!-- deuteranopia colorblind mode -->
          <v-switch
            class="settingsSwitch"
            v-model="settings.blueGreen"
            label="Colorblind mode"
            @change="updateSettings"
          ></v-switch>
          <!-- thick icons -->
          <v-switch
            class="settingsSwitch"
            v-model="settings.useThick"
            label="Use thick icons"
            @change="updateSettings"
          ></v-switch>
          <!-- bold text -->
          <v-switch
            class="settingsSwitch"
            v-model="settings.useBold"
            label="Use bold numbers"
            @change="updateSettings"
          ></v-switch>
          <!-- dark mode -->
          <v-switch
            class="settingsSwitch"
            v-model="settings.darkMode"
            label="Dark mode"
            @change="updateSettings"
          ></v-switch>
        </v-row>
      </v-container>
      <br />
      <!-- accent color -->
      <v-color-picker
        v-model="settings.color"
        dot-size="25"
        swatches-max-height="200"
        @input="updateColor"
      ></v-color-picker>
      <br /><b>Debug</b><br /><br />
      <v-container>
        <v-row>
          <!-- show cell date -->
          <v-switch
            class="settingsSwitch"
            v-model="settings.showCellDate"
            label="Show cell date"
            @change="updateSettings"
          ></v-switch>
          <!-- mark start day -->
          <v-switch
            class="settingsSwitch"
            v-model="settings.markStartDay"
            label="Mark start day"
            @change="updateSettings"
          ></v-switch>
          <!-- start new habits 1 month ago -->
          <v-switch
            class="settingsSwitch"
            v-model="settings.startMonthAgo"
            label="Start new habits 1 month ago"
            @change="updateSettings"
          ></v-switch>
          <!-- show habit weekday and whether used -->
          <v-switch
            class="settingsSwitch"
            v-model="settings.displayWeekday"
            label="Display weekday and whether used"
            @change="updateSettings"
          ></v-switch>
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
          dni z rzędu: {{ getDaysInARow(displayedHabitId) }} <br />
          type: {{ this.habits[displayedHabitId].type }}
          <span v-if="this.habits[displayedHabitId].type == 'numeric'">
            <br />
            daily target: {{ this.habits[displayedHabitId].dailyTarget }}
          </span>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  data: () => ({
    useWeekdays: {
      monday: true,
      tuesday: true,
      wednesday: true,
      thursday: true,
      friday: true,
      saturday: true,
      sunday: true,
    },
    newTaskDate: null,
    newTaskDatePickerMenu: null,
    habitDetailsDialog: false,
    loaded: false,
    habits: [],
    tasks: [],
    displayedHabitId: 0,
    currentDate: new Date(),
    settings: {},
    reloads: {
      table: 1,
    },
  }),
  methods: {
    getIgnoredWeekdaysFromForm() {
      let i = 0;
      let result = [];
      for (let weekday of [
        "sunday",
        "monday",
        "tuesday",
        "wednesday",
        "thursday",
        "friday",
        "saturday",
      ]) {
        if (!this.useWeekdays[weekday]) {
          result.push(i);
        }
        i += 1;
      }
      return result;
    },
    doesHabitUseWeekdayFromDate(habit, date) {
      if ("ignoredWeekdays" in habit) {
        if (habit.ignoredWeekdays.includes(new Date(date).getDay())) {
          return false;
        } else {
          return true;
        }
      }
      return true;
    },
    wasCompletedDuringLastCheckedDays(habit, date) {
      let days = habit.everyNthTime || 1;
      let iteratedDate = new Date(date);
      let back = 0;
      if (habit.type == "boolean") {
        while (back < days) {
          if (
            this.dateToYYYYMMDD(iteratedDate) in habit.records &&
            habit.records[this.dateToYYYYMMDD(iteratedDate)]
          ) {
            return true;
          }
          if (this.doesHabitUseWeekdayFromDate(habit, iteratedDate)) {
            back += 1;
          }
          iteratedDate.setDate(new Date(iteratedDate).getDate() - 1);
        }
      } else if (habit.type == "numeric") {
        while (back < days) {
          if (
            habit.records[this.dateToYYYYMMDD(iteratedDate)] >=
            habit.dailyTarget
          ) {
            return true;
          }
          if (this.doesHabitUseWeekdayFromDate(habit, iteratedDate)) {
            back += 1;
          }
          iteratedDate.setDate(new Date(iteratedDate).getDate() - 1);
        }
      }
      return false;
    },
    getNumericHabitStateStyle(habit, date) {
      let style = "";
      style += "color: ";
      if (habit.records[date] >= habit.dailyTarget) {
        if (this.settings.blueGreen) {
          style += "blue";
        } else {
          style += "green";
        }
      } else {
        if (this.wasCompletedDuringLastCheckedDays(habit, date)) {
          style += "#333";
        } else {
          style += "red";
        }
      }
      style += "; user-select: none; ";
      style += this.settings.useBold ? "font-weight: bold; " : "";
      return style;
    },
    getBooleanHabitStateColor(habit, date) {
      if (!(date in habit.records)) {
        if (this.wasCompletedDuringLastCheckedDays(habit, date)) {
          return "#333";
        } else {
          return "#444444";
        }
      }
      if (date in habit.records && habit.records[date]) {
        return this.settings.blueGreen ? "blue darken-1" : "green darken-2";
      }
      if (date in habit.records && !habit.records[date]) {
        if (this.wasCompletedDuringLastCheckedDays(habit, date)) {
          return "#333";
        } else {
          return "red darken-2";
        }
      }
    },
    getBooleanHabitStateIcon(habit, date) {
      if (!(date in habit.records)) {
        if (this.wasCompletedDuringLastCheckedDays(habit, date)) {
          return "mdi-shield-check-outline";
        }
        // help / checkbox-blank-outline / square-outline / square-rounded-outline ... might be better?
        return "mdi-square-rounded-outline";
      }
      if (date in habit.records && habit.records[date]) {
        return this.settings.useThick ? "mdi-check-bold" : "mdi-check";
      }
      if (date in habit.records && !habit.records[date]) {
        if (this.wasCompletedDuringLastCheckedDays(habit, date)) {
          return "mdi-shield-check-outline";
        }
        return this.settings.useThick ? "mdi-close-thick" : "mdi-window-close";
      }
    },
    getTaskColor(task) {
      let daysLeft = this.timeLeft(task.deadline);
      if (daysLeft < 0) {
        return "background: rgba(255, 0, 0, 0.5);";
      } else if (daysLeft < 2) {
        return "background: rgba(255, 126, 0, 0.5);";
      } else if (daysLeft < 8) {
        return "background: rgba(255, 255, 0, 0.5);";
      } else {
        return "background: rgba(0, 255, 0, 0.5);";
      }
    },
    timeLeft(deadline) {
      let date = new Date(this.dateToYYYYMMDD(new Date()));
      let deadlineDate = new Date(this.dateToYYYYMMDD(deadline));
      let timeLeft = 0;
      if (date < deadlineDate) {
        while (date < deadlineDate && timeLeft < 9999) {
          timeLeft += 1;
          date.setDate(new Date(date).getDate() + 1);
        }
      } else {
        while (date > deadlineDate && timeLeft < 9999) {
          timeLeft -= 1;
          date.setDate(new Date(date).getDate() - 1);
        }
      }
      return timeLeft;
    },
    completedToday(habit) {
      let today = this.dateToYYYYMMDD(new Date());
      if (habit.type == "boolean") {
        console.log("today: " + today);
        return today in habit.records && habit.records[today];
      } else if (habit.type == "numeric") {
        return habit.records[today] >= habit.dailyTarget;
      }
    },
    measureScrollbarWidth() {
      let scrollbox = document.createElement("div");
      scrollbox.style.overflow = "scroll";

      // Append box to document
      document.body.appendChild(scrollbox);

      // Measure inner width of box
      let scrollBarWidth = scrollbox.offsetWidth - scrollbox.clientWidth;
      document.body.removeChild(scrollbox);
      return scrollBarWidth;
    },
    updateWindowSize: function() {
      let content = document.getElementById("content");
      let scale =
        window.innerWidth < 1200 ? (window.innerWidth + 300) / 1500 : 1;
      content.style.transform = "scale(" + scale + ")";
      content.style.margin = `0 0 0 ${-(
        (window.innerWidth -
          window.innerWidth * scale +
          this.measureScrollbarWidth()) /
        2
      )}px`;
      content.style.width = window.innerWidth / scale + "px";
    },
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
    addNewTask: function() {
      this.tasks.push({
        name: document.getElementById("newTaskNameInput").value,
        deadline: this.newTaskDate,
        status: "todo",
        dateCreated: this.dateToYYYYMMDD(new Date()),
      });
      this.refreshAndUpdate();
    },
    addNewBooleanHabit: function() {
      let date = new Date();
      let temp =
        date.getFullYear() +
        "-" +
        (date.getMonth() + (this.settings.startMonthAgo ? 0 : 1)) +
        "-" +
        (date.getDate() > 10 ? date.getDate() : "0" + date.getDate());
      this.habits.push({
        name: document.getElementById("newBooleanHabitNameInput").value,
        startDay: temp,
        records: {},
        type: "boolean",
        everyNthTime: document.getElementById(
          "newBooleanHabitEveryNthTimeInput"
        ).value,
        ignoredWeekdays: this.getIgnoredWeekdaysFromForm(),
      });
      this.refreshAndUpdate();
    },
    addNewNumericHabit: function() {
      let date = new Date();
      let temp =
        date.getFullYear() +
        "-" +
        (date.getMonth() + (this.settings.startMonthAgo ? 0 : 1)) +
        "-" +
        (date.getDate() > 10 ? date.getDate() : "0" + date.getDate());
      this.habits.push({
        name: document.getElementById("newNumericHabitNameInput").value,
        dailyTarget: document.getElementById("newNumericHabitTargetInput")
          .value,
        startDay: temp,
        records: {},
        type: "numeric",
        everyNthTime: document.getElementById(
          "newNumericHabitEveryNthTimeInput"
        ).value,
        ignoredWeekdays: this.getIgnoredWeekdaysFromForm(),
      });
      this.refreshAndUpdate();
    },
    updateSettings: function() {
      localStorage.setItem("settings", JSON.stringify(this.settings));
      this.reloads.table += 1;
    },
    refreshAndUpdate: function() {
      // ugly but helps
      this.settings.hideCompletedHabits = !this.settings.hideCompletedHabits;
      this.settings.hideCompletedHabits = !this.settings.hideCompletedHabits;
      this.settings.hideHabitsCompletedToday = !this.settings
        .hideHabitsCompletedToday;
      this.settings.hideHabitsCompletedToday = !this.settings
        .hideHabitsCompletedToday;
      this.reloads.table += 1;
      localStorage.setItem("habits", JSON.stringify(this.habits));
      localStorage.setItem("tasks", JSON.stringify(this.tasks));
    },
    changeTaskStatus: function(task) {
      if (task.status == "todo") {
        task.status = "done";
      } else {
        task.status = "todo";
      }
      this.refreshAndUpdate();
    },
    changeBooleanStatus: function(habit, date) {
      if (habit.records[date]) {
        habit.records[date] = false;
      } else {
        habit.records[date] = true;
      }
      this.refreshAndUpdate();
    },
    changeNumericStatus: function(habit, date) {
      if (habit.records[date]) {
        habit.records[date] += 1;
      } else {
        habit.records[date] = 1;
      }
      this.refreshAndUpdate();
    },
    dynamicSort: function(property) {
      var sortOrder = 1;
      if (property[0] === "-") {
        sortOrder = -1;
        property = property.substr(1);
      }
      return function(a, b) {
        var result =
          a[property] < b[property] ? -1 : a[property] > b[property] ? 1 : 0;
        return result * sortOrder;
      };
    },
    loadData: function() {
      let storedSettings = localStorage.getItem("settings");
      if (storedSettings) {
        this.settings = JSON.parse(storedSettings);
      } else {
        this.settings.color = "#123456";
        this.settings.blueGreen = false;
        this.settings.useBold = false;
        this.settings.useThick = false;
        this.settings.displayedDays = 8;
        this.settings.daysAgo = 0;
        this.settings.markStartDay = false;
        this.settings.showCellDate = false;
        this.settings.hideCompletedHabits = false;
        this.settings.hideHabitsCompletedToday = false;
        this.settings.hideCompletedTasks = false;
        this.settings.sortTasksByDeadline = false;
        this.settings.startMonthAgo = false;
        this.settings.displayWeekday = false;
        this.settings.darkMode = false;
      }
      let storedHabits = localStorage.getItem("habits");
      if (storedHabits) {
        console.log("found habits data in localStorage!");
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
            records: {
              "2021-10-12": true,
              "2021-10-15": true,
              "2021-10-17": true,
              "2021-10-18": false,
            },
            type: "boolean",
          },
          {
            name: "wejść w pudełko",
            startDay: new Date("2021-10-13"),
            records: {
              "2021-10-13": true,
              "2021-10-14": true,
              "2021-10-15": true,
            },
            type: "boolean",
          },
          {
            name: "zjesc 3 puszki karmy",
            startDay: new Date("2021-11-04"),
            dailyTarget: 3,
            records: {
              "2021-11-04": 2,
              "2021-11-05": 4,
              "2021-11-06": 2,
              "2021-11-07": 5,
              "2021-11-08": 3,
            },
            type: "numeric",
          },
          {
            name: "złapać 2 myszy",
            startDay: new Date("2021-11-06"),
            dailyTarget: 2,
            records: {
              "2021-11-06": 1,
              "2021-11-07": 2,
              "2021-11-08": 1,
            },
            type: "numeric",
          },
          {
            name: "everyNthTime: 2",
            startDay: new Date("2021-11-06"),
            dailyTarget: 2,
            records: {
              "2021-11-06": 1,
              "2021-11-07": 2,
              "2021-11-08": 1,
            },
            type: "numeric",
            everyNthTime: 2,
          },
          {
            name: "not on fridays",
            startDay: new Date("2021-11-06"),
            dailyTarget: 2,
            ignoredWeekdays: [5],
            records: {
              "2021-11-06": 1,
              "2021-11-07": 2,
              "2021-11-08": 1,
            },
            type: "numeric",
            everyNthTime: 2,
          },
          {
            name: "only mon-wed",
            startDay: new Date("2021-11-01"),
            dailyTarget: 2,
            ignoredWeekdays: [0, 4, 5, 6],
            records: {
              "2021-11-06": 1,
              "2021-11-07": 2,
              "2021-11-08": 1,
            },
            type: "numeric",
            everyNthTime: 2,
          },
          {
            name: "only mon-wed-fri-sun",
            startDay: new Date("2021-11-01"),
            dailyTarget: 2,
            ignoredWeekdays: [2, 4, 6],
            records: {
              "2021-11-06": 1,
              "2021-11-07": 2,
              "2021-11-08": 1,
            },
            type: "numeric",
            everyNthTime: 2,
          },
          {
            name: "only mon-wed-fri-sun",
            startDay: new Date("2021-11-01"),
            ignoredWeekdays: [2, 4, 6],
            records: {},
            type: "boolean",
            everyNthTime: 2,
          },
          {
            name: "only mon-wed-fri-sun 3",
            startDay: new Date("2021-11-01"),
            ignoredWeekdays: [2, 4, 6],
            records: {},
            type: "boolean",
            everyNthTime: 3,
          }
        );
      }
      let storedTasks = localStorage.getItem("tasks");
      if (storedTasks) {
        console.log("found tasks data in localStorage!");
        this.tasks = JSON.parse(storedTasks);
        // console.log("from localData:");
        // console.log(this.habits);
        // console.log("length from localData:");
        // console.log(this.habits.length);
        // console.log("stringified from localData: ");
        // console.log(JSON.stringify(this.habits));
      } else {
        this.tasks.push(
          {
            name: "zrobić HLA",
            deadline: "2021-11-11",
            dateCreated: "2021-11-07",
            status: "todo",
          },
          {
            name: "zrobić sprawozdanie elektro",
            deadline: "2021-11-12",
            dateCreated: "2021-11-08",
            status: "todo",
          },
          {
            name: "zrobić pd z matmy",
            deadline: "2021-11-13",
            dateCreated: "2021-11-09",
            status: "todo",
          },
          {
            name: "stare",
            deadline: "2021-11-03",
            dateCreated: "2021-11-09",
            status: "todo",
          },
          {
            name: "nowe",
            deadline: "2021-11-16",
            dateCreated: "2021-11-05",
            status: "todo",
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
            (iDate.getDate() >= 10 ? iDate.getDate() : "0" + iDate.getDate())
        );
        iDate.setDate(iDate.getDate() - 1);
      }
      // console.log(msg);
      // console.log(dates);
      return dates;
    },
    sendDarkModeStateToParent: function(value) {
      this.$emit("input", value);
    },
  },
  mounted() {
    this.loadData();
    this.updateWindowSize();
    window.onresize = this.updateWindowSize;
    //console.log(JSON.stringify(this.habits));
  },
  watch: {
    isDarkModeOn() {
      console.log(this.isDarkModeOn);
      this.sendDarkModeStateToParent(this.isDarkModeOn);
    },
  },
  computed: {
    isDarkModeOn: function() {
      return this.settings.darkMode;
    },
    filteredHabits: function() {
      if (this.settings.hideCompletedHabits) {
        return this.habits.filter(
          (habit) =>
            !this.wasCompletedDuringLastCheckedDays(
              habit,
              this.dateToYYYYMMDD(new Date())
            ) &&
            this.doesHabitUseWeekdayFromDate(
              habit,
              this.dateToYYYYMMDD(new Date())
            )
        );
      }
      if (this.settings.hideHabitsCompletedToday) {
        return this.habits.filter((habit) => !this.completedToday(habit));
      }
      return this.habits;
    },
    filteredTasks: function() {
      let array;
      if (this.settings.hideCompletedTasks) {
        array = this.tasks.filter((task) => task.status != "done");
      } else {
        array = this.tasks;
      }
      if (this.settings.sortTasksByDeadline) {
        array = array.sort(this.dynamicSort("deadline"));
      } else if (!this.settings.sortTasksByDeadline) {
        array = array.sort(this.dynamicSort("dateCreated"));
      }
      return array;
    },
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
  transform-origin: 50vw 0vh;
}

#content.dark {
  --saturation: 73%;
  --value: 50%;
  --value-light: 82%;
  --odd-table-row-background-color: #cccccc;
  --even-table-row-background-color: #c3c3c3;
  --line-between-table-rows-color: #aaaaaa;
  --table-header-text-color: #dddddd;
}

#settingsDiv {
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
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

.settingsSwitch {
  margin: 0px 12px;
}
</style>
