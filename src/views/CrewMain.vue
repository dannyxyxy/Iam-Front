<script>
import { apiClient } from "../utils/axios.js";
import { onMounted, ref } from "vue";
import { CONSTANTS } from "../utils/constants.js";
import commonUtil from "../utils/common-util.js";

export default {
  name: "App",
  computed: {
    CONSTANTS() {
      return CONSTANTS;
    },
  },
  data() {
    return {
      CrewPost: "/CrewPost",
      title: "-",
      summary: "-",
      user: "-",
      isTrendActive: true,
      selected: "trend",
    };
  },
  setup() {
    const boardData = ref({
      crewName: "",
      crewIntro: "",
      crewImg: "",
      ownerInfo: "",
    });
    const getCrewList = async () => {
      await apiClient("crew/getCrewList")
        .then((r) => {
          boardData.value = r.data;
          for (let item in r.data) {
            r.data[item].color = `hsl(${
              parseInt(Math.random() * 24, 10) * 15
            }, 16%, 75%)`;
          }
        })
        .catch((e) => {
          alert("크루 정보를 불러올 수 없습니다.");
          console.log(e);
        });
    };
    const JoinCrew = async (e) => {
      const userLocalInfo = JSON.parse(
        commonUtil.getLocalStorage(CONSTANTS.KEY_LIST.USER_INFO)
      );
      if (userLocalInfo.isMember) {
        alert("이미 크루에 가입된 사용자입니다!");
        return; // Stop further execution of the function
      }
      await apiClient("crew/JoinCrew", {
        userIdx: userLocalInfo.userIdx,
        crewIdx: boardData.crewName,
      })
        .then((r) => {
          alert("크루 가입 성공!");
          location.reload();
        })
        .catch((e) => {
          alert("오류 발생!");
          location.reload();
        });
    };

    onMounted(() => {
      getCrewList();
    });

    return {
      boardData,
      JoinCrew,
    };
  },
};
</script>

<template>
  <div class="crewmain-container">
    <div class="recommended-crews">
      <h2>전체 크루 목록</h2>
      <div class="crew-box-container">
        <div class="crew-box" v-for="item in boardData" :key="item._crewId">
          <router-link :to="{ name: 'CrewPost', query: { id: item._id } }">
            <div
              class="crew-avatar"
              :style="`background-color: ${item.color}`"
            />
          </router-link>
          <div class="crew-info">
            <h3>{{ item.crewName }}</h3>
            <p class="crew-description">{{ item.crewIntro }}</p>
            <button type="submit" class="CrewJoin-button" @click="JoinCrew">
              크루가입
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<style>
@import "../assets/style/view/CrewMain.css";
</style>
