<template>
  <div>
    <el-col style="width: 17.2vw;">
      <div class="div-cls">
        <div class="session-cls" v-for="item in sessionList"
             @click="chooseSession(item)" @contextmenu.prevent="sessionMenu(item)">
          <div class="avatar-cls">
            <el-badge :value="item.unreadCount" :max="99">
              <img class="avatar-img" :src="item.avatar"/>
            </el-badge>
          </div>
          <div class="session-item-cls">
            <span class="session-name-cls">{{ item.sessionName }}</span>
            <span class="session-time-cls">2024-02-02</span>
          </div>
          <div class="session-item-cls">
            <span class="session-msg-cls">哈哈</span>
          </div>
        </div>
      </div>
    </el-col>
  </div>
</template>

<script>
import EventBus from "@/component/event-bus";
import sessionApi from "@/api/session";

export default {
  name: "Session",

  data() {
    return {
      sessionList: [],
    }
  },

  created() {
    this.getSessionList(true);
    EventBus.$on('sendMsgEvent', msg => {
      if (msg.relationId !== '') {
        this.getSessionList(false);
        this.speakNotice('滴滴')
      }
    });
    EventBus.$on('readCount0Event', sessionInfo => {
      this.updateReadTime(sessionInfo.sessionId)
      for (let i = 0; i < this.sessionList.length; i++) {
        if (this.sessionList[i].relationId === sessionInfo.relationId) {
          this.sessionList[i].unreadCount = null;
          break;
        }
      }
    });
  },

  methods: {
    chooseSession(item) {
      EventBus.$emit('sessionInfo', item)
      if (item.unreadCount) {
        this.updateReadTime(item.sessionId)
        item.unreadCount = null
      }
    },

    getSessionList(isFirst) {
      sessionApi.getSessionList().then(res => {
        this.sessionList = res.data.data;
        if (isFirst) {
          EventBus.$emit('sessionList', this.sessionList)
        }
      }).catch(e => {
        this.$message.error(e.data.msg);
      })
    },

    updateReadTime(sessionId) {
      sessionApi.updateReadTime(sessionId)
    },

    speakNotice(msg) {
      const utterThis = new window.SpeechSynthesisUtterance(msg);
      window.speechSynthesis.speak(utterThis);
    },

    sessionMenu(item) {
      if (item.type === 1) {
        this.userSessionMenu();
      }
      if (item.type === 2) {
        this.groupSessionMenu();
      }
    },

    userSessionMenu(event) {
      this.$contextmenu({
        items: [
          {label: "置顶", icon: "el-icon-printer"},
          {label: "标记未读", icon: "el-icon-printer"},
          {label: "消息免打扰", icon: "el-icon-printer"},
          {label: "删除窗口", icon: "el-icon-printer"},
        ],
        event,
        customClass: "custom-class",
        zIndex: 3,
        minWidth: 150
      });
      return false;
    },

    groupSessionMenu(event) {
      this.$contextmenu({
        items: [
          {label: "置顶", icon: "el-icon-printer"},
          {label: "标记未读", icon: "el-icon-printer"},
          {label: "消息免打扰", icon: "el-icon-printer"},
          {label: "消息免打扰", icon: "el-icon-printer"},
          {label: "消息免打扰", icon: "el-icon-printer"},
          {label: "删除窗口", icon: "el-icon-printer"},
        ],
        event,
        customClass: "custom-class",
        zIndex: 3,
        minWidth: 150
      });
      return false;
    },
  }
}
</script>

<style lang="scss" scoped>
.div-cls {
  height: 81vh;
  overflow: auto;
}

// 滚动条样式
.div-cls::-webkit-scrollbar {
  width: 7.8px;
  height: 7.8px;
}

.div-cls::-webkit-scrollbar-thumb {
  border-radius: 10px;
  background: #9e9e9e;
}

.div-cls::-webkit-scrollbar-track {
  border-radius: 10px;
  background: #ededed;
}

// 会话容器
.session-cls {
  height: 7.3vh;
  background-color: $window-item-color;
  border-radius: 6px;
  margin-bottom: 2px;
  cursor: pointer;
}

.session-cls:hover {
  background-color: $logo-color;
}

// 会话对象
.session-item-cls {
  height: 48%;
  width: 74%;
  display: inline;
  float: left;
}

.avatar-cls {
  height: 100%;
  width: 26%;
  float: left;
}

.avatar-img {
  width: 3.4vw;
  height: 7.3vh;
  border-radius: 6px;
}

.session-name-cls {
  width: 58%;
  font-size: 13.6px;
  display: inline-block;
  margin-top: 3%;
  color: $session-nick-color;
}

.session-time-cls {
  width: 40%;
  font-size: 12px;
  display: inline-block;
  color: $session-msg-color;
}

.session-msg-cls {
  width: 90%;
  font-size: 13px;
  display: inline-block;
  color: $session-msg-color;
}
</style>
