<template>
  <table>
    <tbody>
      <tr @mouseleave="mouseExit">
        <td
          v-bind:class="{
            'has-background-primary has-text-white' : selectedDates.includes(index), 
            'has-background-grey-lighter' : tableData[index].publicHoliday}"
          @mousedown="dateClicked(index)"
          @mouseover="dateContinued(index)"
          @mouseup="mouseReleased"
          class="day"
          v-for="(day, index) in tableData"
          :key="index"
        >
          <span>{{day.number}}</span>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script>
import {
  getDaysInMonth,
  startOfMonth,
  endOfMonth,
  eachDay,
  format,
  addDays,
  getYear
} from "date-fns";
import db from "@/firebaseInit";
import { mapState } from "vuex";
export default {
  data() {
    return {
      selectedDates: []
    };
  },
  methods: {
    dateClicked(itemIndex) {
      this.$store.state.mouseCurrentlyDown = true;
      this.selectedDates.push(itemIndex);
    },
    dateContinued(itemIndex) {
      if (this.$store.state.mouseCurrentlyDown == true) {
        if (itemIndex != this.selectedDates[this.selectedDates.length - 1]) {
          if (
            itemIndex - this.selectedDates[this.selectedDates.length - 1] >
            1
          ) {
            const temp = [];
            for (var i = this.selectedDates[0]; i < itemIndex; i++) {
              temp.push(i);
            }
            this.selectedDates = temp;
          }
          if (
            itemIndex - this.selectedDates[this.selectedDates.length - 1] <
            -1
          ) {
            const temp = [];
            for (var i = this.selectedDates[0]; i > itemIndex; i--) {
              temp.push(i);
            }
            this.selectedDates = temp;
          }
          if (itemIndex == this.selectedDates[this.selectedDates.length - 2]) {
            this.selectedDates.pop();
            this.selectedDates.pop();
          }
          this.selectedDates.push(itemIndex);
        }
      }
    },
    mouseReleased() {
      this.$store.state.mouseCurrentlyDown = !this.$store.state
        .mouseCurrentlyDown;
      const dates = [
        this.selectedDates[0] + 1,
        this.selectedDates[this.selectedDates.length - 1] + 1
      ];
      this.$store.state.leaveStartDate = format(
        new Date(this.currentYear, this.selectedMonth, Math.min(...dates)),
        "YYYY-MM-DD"
      );
      this.$store.state.leaveEndDate = format(
        new Date(this.currentYear, this.selectedMonth, Math.max(...dates)),
        "YYYY-MM-DD"
      );
      if (this.selectedDates.length > 0) {
        this.$store.state.leaveModalActive = true;
      }
      this.selectedDates = [];
    },
    mouseExit() {
      this.selectedDates = [];
      this.$store.state.mouseCurrentlyDown = false;
    },
    updateTable() {
      console.log(this.userid);
      this.$store.state.tableData = [];
      const days = [];
      const monthStartDate = startOfMonth(
        new Date(this.currentYear, this.selectedMonth)
      );
      const monthEndDate = endOfMonth(
        new Date(this.currentYear, this.selectedMonth)
      );
      const monthDays = eachDay(monthStartDate, monthEndDate);
      monthDays.forEach(function(day) {
        days.push({ number: format(`${day}`, "D") });
      });
      this.$store.state.tableData.push(...days);
    }
  },
  created() {
    this.updateTable();
  },
  watch: {
    selectedMonth() {
      this.updateTable();
    }
  },
  computed: {
    ...mapState(["tableData"]),
    isUserId(num) {
      if (this.tableData[index].holidays == userid) {
        return true;
      }
    }
  },
  props: ["currentYear", "selectedMonth", "userid"]
};
</script>

<style>
#monthDays div table tbody td {
  width: 30px !important;
}
.day {
  width: 30px !important;
  min-width: 30px !important;
  max-width: 30px !important;
  text-align: center;
  padding: 0.5rem;
  /* background: rgba(150, 150, 150, 0.1); */
  /* border: 1px solid rgba(150, 150, 150, 0.1); */
  -webkit-touch-callout: none; /* iOS Safari */
  -webkit-user-select: none; /* Safari */
  -khtml-user-select: none; /* Konqueror HTML */
  -moz-user-select: none; /* Firefox */
  -ms-user-select: none; /* Internet Explorer/Edge */
  user-select: none; /* Non-prefixed version, currently
                                  supported by Chrome and Opera */
}
.day:hover {
  cursor: pointer;
  background: rgba(150, 150, 150, 0.1);
}
</style>