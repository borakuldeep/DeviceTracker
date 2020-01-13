<template>
  <div class="enternoti">
    <div class="heading">Notifications Panel</div>
    <div v-for="(item, index) in selectDevices" :key="index">
      <div class="list" v-bind:class="[item.type]">{{ item.text }}</div>
    </div>
  </div>
</template>

<script>
//import ioClient from "socket.io-client";
import client from "./Connection.vue";

export default {
  props: ["devices"],
  data() {
    return {
      notifications: []
    };
  },
  created() {
    //let client = ioClient.connect("http://localhost:3000");
    client.on("notifications", noti => {
      this.notifications = [...noti];
      //console.log("notification arrived: ", noti);
      //console.log("noti props: ", this.devices);
    });
  },
  computed: {
    selectDevices() {
      let noti = [];
      this.notifications.forEach(item => {
        if (item.type == "entered") {
          this.devices.forEach(device => {
            if (item.device == device) noti.push(item);
          });
        } else noti.push(item);
      });
      //console.log("Seleced Noti: ", noti);
      return noti;
    }
  }
};
</script>

<style scoped>
.enternoti {
  --visibility: hidden;
  position: absolute;
  width: 380px;
  height: 100vh;
  border: 1px solid black;
  border-radius: 10px;
  font-size: 1rem;
  font-weight: 500;
  --opacity: 40%;
  z-index: 1000;
  margin-left: 80.5%;
  opacity: 80%;
  --overflow-y: scroll;
  box-sizing: border-box;
}

.list {
  width: 300px;
  margin: 10px;
  height: 50px;
  --font-weight: bold;
  --background: white;
  margin: 3px;
  font-size: 1.1rem;
  margin-top: 10px;
  margin-left: 20px;
  padding: 5px;
  color: white;
  border: 1px solid white;
  border-radius: 25px;
  border-bottom-right-radius: 1px;
}

.Proximity {
  background: orangered;
  text-align: center;
  animation: pop 0.5s;
}

.entered {
  background: #4d79ff;
  color: white;
  text-align: center;
  animation: pop 0.5s;
}

@keyframes pop {
  from {
    transform: scale(0, 0);
  }
  to {
    transform: scale(1, 1);
  }
}

.heading {
  text-align: center;
  font-weight: bold;
  color: white;
  background: black;
  height: 30px;
  width: 300px;
  margin: 6px;
  margin-left: 30px;
  padding-top: 3px;
  border-radius: 10px;
  box-sizing: border-box;
}
</style>
