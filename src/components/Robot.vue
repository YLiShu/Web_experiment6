<template>
  <div class="wrap">
    <div class="header">
      <h3>SmartChat</h3>
      <img src="/src/assets/robot.png" alt="icon" />
    </div>
    <div class="main" ref="chatMain">
      <ul class="talk_list">
        <li v-for="(msg, index) in messages" :key="index" :class="msg.type">
          <img :src="msg.icon" />
          <span>{{ msg.content }}</span>
        </li>
      </ul>
    </div>
    <div class="footer">
      <img src="/src/assets/user.png" alt="icon" />
      <input
        type="text"
        placeholder="说点什么吧..."
        class="input_txt"
        v-model="inputMsg"
        @keydown.enter="sendMessage"
        :disabled="isInputDisabled"
      />
      <input
        id="btnSend"
        type="button"
        value="发 送"
        class="input_sub"
        @click="sendMessage"
      />
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref, nextTick } from "vue";
import robotIcon from "../assets/robot.png";
import userIcon from "../assets/user.png";

const isInputDisabled = ref(false);
const chatMain = ref(null);
const key = import.meta.env.VITE_KEY;
const messages = ref([
  {
    type: "left_word",
    icon: robotIcon,
    content: "嗨~，来和我聊天吧！",
  },
]);

const inputMsg = ref("");

const sendMessage = () => {
  const text = inputMsg.value.trim();
  if (text === "/clear") {
    messages.value = [];
    inputMsg.value = "";
  } else if (text !== "") {
    messages.value.push({
      type: "right_word",
      icon: userIcon,
      content: text,
    });
    inputMsg.value = "";
    nextTick(() => resetUI());
    postChatGpt(text);
  }
};

const postChatGpt = (text) => {
  const data = {
    model: "gpt-3.5-turbo",
    messages: [{ role: "user", content: text }],
  };
  isInputDisabled.value = true;
  inputMsg.value = "SmartChat思考中...";
  axios
    .post("https://api.chatanywhere.com.cn/v1/chat/completions", data, {
      headers: {
        Authorization: `Bearer ${key}`,
      },
    })
    .then((response) => {
      const res = response.data;
      if (res.id) {
        const msg = res.choices[0].message.content;
        messages.value.push({
          type: "left_word",
          icon: robotIcon,
          content: msg,
        });
        nextTick(() => resetUI());
        isInputDisabled.value = false;
        inputMsg.value = "";
      }
    })
    .catch((error) => {
      console.log(error);
      isInputDisabled.value = true;
      inputMsg.value = "";
    });
};
const resetUI = () => {
  chatMain.value.scrollTop = chatMain.value.scrollHeight;
};
</script>

<style>
.wrap {
  position: fixed;
  width: 450px;
  left: 50%;
  margin-left: -225px;
  top: 20px;
  bottom: 20px;
  border: 1px solid #ebebeb;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 0 30px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}
.header {
  height: 55px;
  background: linear-gradient(
    90deg,
    rgba(246, 60, 47, 0.6),
    rgba(128, 58, 242, 0.6)
  );
  overflow: hidden;
  display: flex;
  width: 100%;
  justify-content: space-between;
  align-items: center;
}
.header h3 {
  color: #faf3fc;
  line-height: 55px;
  font-weight: normal;
  letter-spacing: 2px;
  margin-left: 25px;
  font-size: 18px;
  text-shadow: 0px 0px 5px #944846;
}
.header img {
  margin-right: 25px;
  width: 40px;
  height: 40px;
}
.main {
  position: absolute;
  left: 0;
  right: 0;
  top: 55px;
  bottom: 55px;
  background-color: #f4f3f3;
  box-sizing: border-box;
  padding: 10px 0;
  overflow: auto;
}
.talk_list {
  position: absolute;
  width: 100%;
  left: 0px;
  top: 0px;
}
.talk_list li {
  overflow: hidden;
  margin: 20px 0px 30px;
}
.talk_list .left_word img {
  float: left;
  margin-left: 20px;
  width: 40px;
  height: 40px;
}
.talk_list .left_word span {
  float: left;
  background-color: #fe9697;
  padding: 10px 15px;
  max-width: 290px;
  border-radius: 12px;
  font-size: 16px;
  color: #fff;
  margin-left: 13px;
  position: relative;
  line-height: 24px;
}
.talk_list .left_word span:before {
  content: "";
  position: absolute;
  left: -8px;
  top: 3px;
  width: 13px;
  height: 12px;
}
.talk_list .right_word img {
  float: right;
  margin-right: 20px;
  width: 40px;
  height: 40px;
}
.talk_list .right_word span {
  float: right;
  background-color: #fff;
  padding: 10px 15px;
  max-width: 290px;
  border-radius: 12px;
  font-size: 16px;
  color: #000;
  margin-right: 13px;
  position: relative;
  line-height: 24px;
}
.talk_list .right_word span:before {
  content: "";
  position: absolute;
  right: -8px;
  top: 3px;
  width: 13px;
  height: 12px;
}

.footer {
  width: 100%;
  height: 55px;
  left: 0px;
  bottom: 0px;
  background-color: #fff;
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
}

.footer img {
  width: 40px;
  height: 40px;
}

.input_txt {
  width: 270px;
  height: 37px;
  border: 0px;
  background-color: #f4f3f3;
  margin: 9px 0 0 20px;
  border-radius: 8px;
  padding: 0px;
  outline: none;
  text-indent: 15px;
}
.input_sub {
  width: 70px;
  height: 37px;
  border: 0px;
  background-color: #fe9697;
  margin: 9px 0 0 15px;
  border-radius: 8px;
  padding: 0px;
  outline: none;
  color: #fff;
  cursor: pointer;
}
</style>
