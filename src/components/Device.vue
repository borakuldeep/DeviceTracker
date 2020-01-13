<template>
  <div class="devices" id="device-panel">
    <div v-for="(item, index) in devices" :key="index">
      <button
        class="device-btn"
        :class="[item.class]"
        @click="toggle(item.name, item.class)"
      >
        {{ item.name }}
      </button>
    </div>
    <button @click="removeAllAreas" class="remove-areas">Remove Areas</button>
    <div class="developer">Developer: Kuldeep Bora</div>
  </div>
</template>

<script>
import client from "./Connection.vue";
import Swal from "sweetalert2";

export default {
  data() {
    return {
      devices: [
        { name: "Device 1", status: "Active", class: "device1" },
        { name: "Device 2", status: "Active", class: "device2" },
        { name: "Device 3", status: "Active", class: "device3" },
        { name: "Device 4", status: "Active", class: "device4" },
        { name: "Device 5", status: "Active", class: "device5" }
      ]
    };
  },
  mounted() {
    client.on("connect", function() {
      let buttons = [...document.getElementsByClassName("device-btn")];
      buttons.forEach(btn => {
        btn.disabled = false;
      });
    });
    client.on("disconnect", function() {
      //console.log(buttons);
      let buttons = [...document.getElementsByClassName("device-btn")];
      buttons.forEach(btn => {
        btn.disabled = true;
      });
    });
  },
  updated() {
    /* console.log("tracking props: ", this.tracking);
    let btn = document.getElementsByClassName("device-btn");
    if (this.tracking == false) {
      btn[0].classList.add("inactive");
    } else btn[0].classList.remove("inactive"); */
  },
  methods: {
    toggle(name, classname) {
      this.$emit("toggle", name);
      //console.log("toggle of ", classname);
      let btn = document.getElementsByClassName(classname);
      if (btn[0].classList.contains("inactive")) {
        btn[0].classList.remove("inactive");
      } else btn[0].classList.add("inactive");
    },
    removeAllAreas() {
      client.emit("remove-areas", {});
      Swal.fire("Delete request sent", "", "success");
    }
  }
};
</script>

<style scoped>
.devices {
  --visibility: hidden;
  display: flex;
  position: absolute;
  width: 100%;
  height: 70px;
  background: white;
  --color: white;
  --box-shadow: 5px 3px 2px #888888;
  --font-size: 10rem;
  --opacity: 40%;
  z-index: 900;
  margin-top: 93vh;
  --opacity: 75%;
  --overflow-y: scroll;
  box-sizing: border-box;
}

.device-btn {
  border-radius: 10px;
  z-index: 600;
  width: 120px;
  height: 40px;
  margin: 15px;
  --background-color: grey;
  color: white;
  background: #4d79ff;
  box-sizing: border-box;
  font-size: 1.2rem;
  font-weight: bold;
  --box-shadow: 5px 3px 2px #888888;
}

.remove-areas {
  border-radius: 10px;
  z-index: 600;
  width: 100px;
  height: 40px;
  margin: 15px;
  margin-left: 23%;
  color: white;
  background: orange;
  box-sizing: border-box;
  font-size: 0.9rem;
  font-weight: bold;
  box-shadow: 5px 3px 2px #888888;
}

.device1 {
  --background-color: chartreuse;
  --border: 1px solid chartreuse;
}

.device2 {
  --border: 1px solid orangered;
}

.device3 {
  --border: 1px solid blueviolet;
}

.device4 {
  --border: 1px solid yellow;
}

.device5 {
  --border: 1px solid mediumvioletred;
}

.inactive {
  background: grey;
  --opacity: 0.6;
}

.developer {
  font-size: 0.9rem;
  margin: 25px;
  margin-left: 10%;
  z-index: 600;
}
</style>
