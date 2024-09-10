<template>
  <article>
    <header>
      <div class="left">
        <img :src="location" alt="" /><span> 时间线 </span>
      </div>
      <div class="right">
        <img :src="lessen" alt="" />
        <img :src="close" alt="" />
      </div>
    </header>
    <div class="center">
      <div class="control">
        <div class="start">
          <img :src="startImg" alt="" @click="startClick" />
          <img :src="stopImg" alt="" @click="stopClick" />
        </div>
        <div class="speed">
          <button @click="speedClick('cut')">
            <img :src="decelerate" alt="" />
            <span>速度</span>
          </button>
          <button @click="speedClick()">速度重置</button>
          <button @click="speedClick('up')">
            <span>速度</span>
            <img :src="accelerate" alt="" />
          </button>
        </div>
        <div class="time">
          <div class="ball"></div>
          <img :src="calendar" alt="" />
          <input
            type="datetime-local"
            v-model="dateValue"
            v-show="inputType"
            disabled
          />
          <input type="date" v-model="dateValue" v-show="!inputType" disabled />
          <span @click="currentClick">当前时间</span>
        </div>
      </div>
      <div class="show">
        <div class="dimensionality">
          <div
            v-for="(item, index) in dimensionalityList"
            :key="index"
            :class="{ active: dimensionalityActive === item }"
            @click="dimensionalityClick(item)"
          >
            {{ item }}
          </div>
        </div>
        <div
          class="timeLine"
          ref="dayDrag"
          @mousedown="startDrag"
          @mousemove="handleDrag"
          @mouseleave="endDrag"
          @mouseup="endDrag"
        >
          <div
            class="each"
            v-for="(item, index) in currentList"
            :key="index"
            @click="dragClick(item.date, index)"
          >
            <span>{{ currentValue(item.date) }} </span>
          </div>

          <div class="line"></div>
        </div>
      </div>
    </div>
  </article>
</template>

<script setup lang="ts">
import { computed, onMounted, ref, watch } from "vue";
import { dayjs } from "element-plus";
import location from "@/assets/images/location.png";
import lessen from "@/assets/images/lessen.png";
import close from "@/assets/images/close.png";
import startImg from "@/assets/images/start.png";
import stopImg from "@/assets/images/stop.png";
import accelerate from "@/assets/images/accelerate.png";
import decelerate from "@/assets/images/decelerate.png";
import calendar from "@/assets/images/calendar.png";

// 速度逻辑
const timer = ref<any>(null);
const speed = ref(1000);
const speedClick = (status = "") => {
  if (status === "up") {
    if (speed.value === 10) return;
    speed.value /= 10;
  } else if (status === "cut") {
    if (speed.value === 1000) return;
    speed.value *= 10;
  } else {
    speed.value = 1000;
  }
};

// 更改维度逻辑
const dimensionalityList = ref(["年制", "月制", "天制", "时制", "分制"]);
const dimensionalityActive = ref("天制");
const dimensionalityClick = (item: any) => {
  dimensionalityActive.value = item;
  clearInterval(timer.value);
  timer.value = null;
  speed.value = 1000;
  setTimeout(() => {
    currentClick();
  }, 10);
};
const inputType = computed(() => {
  if (
    dimensionalityActive.value === "时制" ||
    dimensionalityActive.value === "分制"
  ) {
    return true;
  }
  return false;
});

// 日期相关逻辑
const dateValue = ref("");
// 当天日期
const today = ref(dayjs().format("YYYY-MM-DDTHH:mm"));
// 当月月份
const tomonth = ref(dayjs().format("YYYY-MM-DD"));
// 所有天数
const dayList = ref(getDays());
// 所有月份
const monthList = ref(getMonths());
// 所有年份
const yearList = ref(getYears());
// 小时
const hourList = ref(
  Array.from({ length: 24 }, (_, i) => ({
    date: i.toString().padStart(2, "0"),
  }))
);
// 分钟
const minuteList = ref(
  Array.from({ length: 60 }, (_, i) => ({
    date: i.toString().padStart(2, "0"),
  }))
);
// 当前选中Index
const currentIndex = ref(
  dayList.value.findIndex((f) => f.date === dayjs().format("YYYY-MM-DD"))
);
// 修改当前选中Index
const setCurrentIndex = () => {
  switch (dimensionalityActive.value) {
    case "年制":
      currentIndex.value = yearList.value.findIndex(
        (f) => f.date === dayjs().format("YYYY")
      );
      return;
    case "月制":
      currentIndex.value = monthList.value.findIndex(
        (f) => f.date === dayjs().format("YYYY-MM")
      );
      return;
    case "天制":
      currentIndex.value = dayList.value.findIndex(
        (f) => f.date === dayjs().format("YYYY-MM-DD")
      );
      return;
    case "时制":
      currentIndex.value = hourList.value.findIndex(
        (f) => f.date === dayjs().format("HH")
      );
      return;
    case "分制":
      currentIndex.value = hourList.value.findIndex(
        (f) => f.date === dayjs().format("mm")
      );
      return;
    default:
      currentIndex.value = dayList.value.findIndex(
        (f) => f.date === dayjs().format("YYYY-MM-DD")
      );
      return;
  }
};
// 当前维度数组
const currentList = computed(() => {
  switch (dimensionalityActive.value) {
    case "年制":
      return yearList.value;
    case "月制":
      return monthList.value;
    case "天制":
      return dayList.value;
    case "时制":
      return hourList.value;
    case "分制":
      return minuteList.value;
    default:
      return dayList.value;
  }
});
// 当前展示value
const currentValue = (item: any) => {
  switch (dimensionalityActive.value) {
    case "年制":
      return item;
    case "月制":
      return item.split("-")[1];
    case "天制":
      return item.split("-")[2];
    case "时制":
      return item;
    case "分制":
      return item;
    default:
      return item.split("-")[2];
  }
};
// 点击某一天更改dateValue
const dateValueChange = (item: any) => {
  console.log(item, "item");
  switch (dimensionalityActive.value) {
    case "年制":
      dateValue.value = `${item}-01-01`;
      return;
    case "月制":
      dateValue.value = `${item}-01`;
      return;
    case "天制":
      dateValue.value = item;
      return;
    case "时制":
      dateValue.value = `${tomonth.value}T${item}:00`;
      return;
    case "分制":
      dateValue.value = `${dayjs().format("YYYY-MM-DDTHH")}:${item}`;
      return;
    default:
      return dayList.value;
  }
};
// 回到当前天
const currentClick = () => {
  setCurrentIndex();
  dayDrag.value.scrollLeft = 60 * currentIndex.value - 439.4;
  (document.querySelector(".line") as HTMLElement).style.left = `${
    60 * currentIndex.value
  }px`;
  if (inputType.value) {
    dateValue.value = today.value;
  } else {
    dateValue.value = tomonth.value;
  }
  clearInterval(timer.value);
  timer.value = null;
};
// 点击某一天
const dragClick = (item: string, index: number) => {
  if (timer.value) return;
  currentIndex.value = index;
  dayDrag.value.scrollLeft = 60 * currentIndex.value - 439.4;
  (document.querySelector(".line") as HTMLElement).style.left = `${
    60 * currentIndex.value
  }px`;
  dateValueChange(item);
};
const startClick = () => {
  if (timer.value) return;
  timer.value = setInterval(() => {
    const left = (document.querySelector(".line") as HTMLElement).offsetLeft;
    if (left / 60 === currentList.value.length) {
      clearInterval(timer.value);
      timer.value = null;
      return;
    }
    if (
      (document.querySelector(".line") as HTMLElement).getBoundingClientRect()
        .x > 800
    ) {
      dayDrag.value.scrollLeft += 700;
    }
    if (left % 60 === 0) {
      currentIndex.value = left / 60;
      dateValueChange(currentList.value[currentIndex.value].date);
    }
    (document.querySelector(".line") as HTMLElement).style.left = `${
      left + 1
    }px`;
  }, speed.value);
};
const stopClick = () => {
  clearInterval(timer.value);
  timer.value = null;
};

watch(
  () => speed.value,
  () => {
    if (timer.value) {
      clearInterval(timer.value);
      timer.value = setInterval(() => {
        const left = (document.querySelector(".line") as HTMLElement)
          .offsetLeft;
        if (left / 60 === currentList.value.length) {
          clearInterval(timer.value);
          timer.value = null;
          return;
        }
        if (
          (
            document.querySelector(".line") as HTMLElement
          ).getBoundingClientRect().x > 800
        ) {
          dayDrag.value.scrollLeft += 700;
        }
        if (left % 60 === 0) {
          currentIndex.value = left / 60;
          dateValueChange(currentList.value[currentIndex.value].date);
        }
        (document.querySelector(".line") as HTMLElement).style.left = `${
          left + 1
        }px`;
      }, speed.value);
    }
  }
);

onMounted(() => {
  currentClick();
});

// 获取日期逻辑
function getDays() {
  const currentDate = dayjs();
  const currentYear = currentDate.year();
  const result = [];

  // 循环获取前后一年的数据
  for (let yearOffset = -1; yearOffset <= 1; yearOffset++) {
    const year = currentYear + yearOffset;

    // 获取每个月的数据
    for (let month = 1; month <= 12; month++) {
      const numDays = dayjs(new Date(year, month, 1)).subtract(1, "day").date();

      const monthlyData = Array.from({ length: numDays }, (_, i) => ({
        date: `${year}-${String(month).padStart(2, "0")}-${String(
          i + 1
        ).padStart(2, "0")}`,
      }));

      result.push(...monthlyData);
    }
  }

  return result;
}
function getMonths() {
  const currentDate = dayjs();
  const currentYear = currentDate.year();
  const currentMonth = currentDate.month() + 1;

  const result = [];

  // 循环获取前后10年的数据
  for (let yearOffset = -10; yearOffset <= 10; yearOffset++) {
    const year = currentYear + yearOffset;

    // 获取当前年份的月份数据
    if (year === currentYear) {
      for (let month = currentMonth; month <= 12; month++) {
        const paddedMonth = String(month).padStart(2, "0");
        result.push({ date: `${year}-${paddedMonth}` });
      }
    } else {
      // 获取其他年份的所有月份数据
      for (let month = 1; month <= 12; month++) {
        const paddedMonth = String(month).padStart(2, "0");
        result.push({ date: `${year}-${paddedMonth}` });
      }
    }
  }

  return result;
}
function getYears() {
  const currentDate = new Date();
  const currentYear = currentDate.getFullYear();
  const result = [];

  for (let i = currentYear - 50; i <= currentYear + 50; i++) {
    result.push({ date: `${i}` });
  }

  return result;
}
// 按住滚动逻辑
const start = ref(false);
const startX = ref(0);
const scrollLeft = ref(0);
const dayDrag = ref<any>(null);
function startDrag(e: any) {
  start.value = true;
  startX.value = e.pageX - dayDrag.value.offsetLeft;
  scrollLeft.value = dayDrag.value.scrollLeft;
  dayDrag.value.style.cursor = "grabbing";
}
function handleDrag(e: any) {
  if (!start.value) return;
  e.preventDefault();
  const x = e.pageX - dayDrag.value.offsetLeft;
  const dragDistance = (x - startX.value) * 2;
  dayDrag.value.scrollLeft = scrollLeft.value - dragDistance;
}
function endDrag() {
  start.value = false;
  dayDrag.value.style.cursor = "grab";
}
</script>

<style scoped lang="scss">
article {
  user-select: none;
  width: 926px;
  font-family: SimHei;
  position: absolute;
  top: 0;
  left: 0;
  header {
    height: 32px;
    width: 926px;
    background-image: url("@/assets/images/longTop.png");
    background-repeat: no-repeat;
    background-size: 100%;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 8px 0 22px;
    .left {
      display: flex;
      align-items: center;
      height: 100%;
      img {
        width: 15px;
        height: 15px;
      }
      span {
        margin-left: 7px;
        font-family: MicrosoftYaHei;
        font-size: 16px;

        color: #ffffff;
      }
    }
    .right {
      display: flex;
      align-items: center;
      height: 100%;
      gap: 0 5px;
      img {
        cursor: pointer;
      }
    }
  }
  .center {
    width: 100%;
    padding: 8px 22px 18px;
    height: 223px;
    overflow: hidden;
    background-color: rgba(6, 72, 143, 0.8);
    border: 2px solid #0e4c95;
    border-image: linear-gradient(180deg, #0b3772 0%, #0e4d96 100%) 2;
    background-image: url("@/assets/images/longBottom.png");
    background-repeat: no-repeat;
    background-position: 0 100%;
    background-size: 100%;
    .control {
      width: 100%;
      height: 58px;
      display: flex;
      align-items: center;
      border-bottom: 1px solid #0c69cd;
      position: relative;
      &::before,
      &::after {
        content: "";
        position: absolute;
        width: 10px;
        height: 1px;
        background: #10b3ff;
        bottom: -1px;
      }
      &::before {
        left: 0;
      }
      &::after {
        right: 0;
      }
      .start {
        width: 72px;
        height: 26px;
        background-image: url("@/assets/images/startBox.png");
        background-repeat: no-repeat;
        background-size: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        margin-left: 10px;
        img {
          cursor: pointer;
        }
        img:nth-child(1) {
          margin-right: 12px;
        }
      }
      .speed {
        display: flex;
        justify-content: space-between;
        align-items: center;
        width: 220px;
        height: 20px;
        margin-left: 26px;
        button {
          cursor: pointer;
          font-family: SimHei;
          font-size: 12px;
          line-height: 16px;
          color: #ffffff;
          border-radius: 4px;
          border: 1px solid #16bdff;
          background: #1376cc;
          display: flex;
          justify-content: center;
          align-items: center;
          img:nth-child(1) {
            margin-right: 4px;
          }
          img:nth-child(2) {
            margin-left: 4px;
          }
        }
      }
      .time {
        width: 290px;
        height: 28px;
        margin-left: 56px;
        display: flex;
        align-items: center;
        position: relative;
        .ball {
          margin-right: 8px;
          width: 8px;
          height: 8px;
          background: radial-gradient(
            50% 50% at 50% 50%,
            #ffffff 0%,
            #08ffff 100%
          );
          border-radius: 50%;
          box-shadow: 0px 0px 7px 0px #08ffff;
        }
        img {
          position: absolute;
          right: 65px;
          top: 5px;
          pointer-events: none;
        }
        input {
          font-family: SimHei;
          font-size: 17px;
          color: #fff;
          padding: 0 3px 0 7px;
          width: 214px;
          height: 100%;
          border-radius: 4px;
          background: rgba(0, 25, 74, 0.6);
          border: 1px solid #0080fe;
          &::-webkit-calendar-picker-indicator {
            opacity: 0;
          }
        }
        span {
          cursor: pointer;
          margin-left: 12px;
          color: #ffffff;
          text-decoration: underline;
          font-size: 12px;
        }
      }
    }
    .show {
      margin-top: 12px;
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      .dimensionality {
        width: 468px;
        height: 32px;
        background: #012f76;
        display: flex;
        justify-content: center;
        align-items: center;
        border-width: 1px 0px 1px 0px;
        border-style: solid;
        border-color: #2594ba;
        div {
          height: 100%;
          flex: 1;
          display: flex;
          justify-content: center;
          align-items: center;
          cursor: pointer;
          font-size: 14px;
          color: #bbbbbb;

          &.active {
            color: #ffffff;
            background: linear-gradient(
              270deg,
              rgba(0, 149, 255, 0) 0%,
              #0084ff 50%,
              rgba(0, 149, 255, 0) 100%
            );
          }
        }
      }
    }
    .timeLine {
      width: 100%;
      height: 96px;
      position: relative;
      display: flex;
      overflow: hidden;
      background-image: url("@/assets/images/line_bg.png");
      background-repeat: no-repeat;
      background-size: 100%;
      background-position: 0 11px;

      .each {
        position: relative;
        margin-top: 19.7px;
        width: 60px;
        flex: 0 0 auto;
        display: flex;
        flex-direction: column;
        height: 50px;
        border-left: 1px solid #0084ff;
        span {
          position: absolute;
          bottom: -26px;
          left: -4px;
          font-family: Microsoft YaHei;
          font-size: 12px;
          color: #fff;
        }
      }
      .line {
        background: #00c8ff;
        width: 2px;
        height: 66px;
        position: absolute;
        left: 0px;
        top: 12px;
        &::before {
          content: "";
          position: absolute;
          left: -6.1px;
          top: -6.5px;
          width: 0;
          height: 0;
          border: 0 solid transparent;
          border-left: calc(36.5px / 5) solid transparent;
          border-right: calc(36.5px / 5) solid transparent;
          border-top: calc(35px / 5) solid #00c8ff;
        }
      }
    }
  }
}

::-webkit-scrollbar {
  width: 2px;
}
</style>
