<template>
  <div class="mapcont" id="map">
    <div class="header">
      TRG Tracker
      <div class="status" id="server" />
    </div>
    <div class="menu">Help!</div>
    <transition name="fade">
      <div class="showinfo" v-show="info">
        Functionality:
        <li>
          Right click anywhere on the map to create area block.
          <p>
            Area will be created using right click point as bottom left corner.
          </p>
        </li>

        <li>
          Toogle device track/untrack using device panel buttons at footer.
        </li>
        <li>
          Press start tracking button (purple) to start the tracking. Button
          will disable once tracking starts to prevent more than one setInterval
          calls in backend. :/
        </li>
        <li>
          Press reset button (blue) to reset devices to original positions.
        </li>
        <li>
          Notifications about devices updates will be shown on right panel.
        </li>
        <li>
          Status light on TRG header will turn green/gray on server connection.
        </li>
        <br />
      </div>
    </transition>
    <div class="info" id="info">Right click to create area !</div>
    <button id="reset-btn" class="reset-btn" @click="resetLocations">
      reset devices location <br />(takes upto 5 sec)
    </button>
    <button class="start-btn" id="start" @click="startTracking">
      Start Tracking<br />(takes 5 sec)
    </button>
    <Notification :devices="deviceNames" />
    <DevicePanel @toggle="toggleTracking" />
  </div>
</template>

<script>
import Swal from "sweetalert2";
import Notification from "./NotificationPanel.vue";
import DevicePanel from "./Device.vue";
import client from "./Connection.vue";

let mymap;

export default {
  data() {
    return {
      deviceList: [
        /* { name: "Device 1", status: "Active", handle: d1 },
        { name: "Device 2", status: "Active", handle: d2 },
        { name: "Device 3", status: "Active", handle: d3 },
        { name: "Device 4", status: "Active", handle: d4 },
        { name: "Device 5", status: "Active", handle: d5 } */
      ],
      areas: [],
      info: false,
      count: 0,
      rect: []
    };
  },
  computed: {
    deviceNames() {
      let dn = [];
      this.deviceList.forEach(item => {
        if (item.status == "Active") dn.push(item.name);
      });
      return dn;
    }
  },
  components: {
    Notification,
    DevicePanel
  },
  mounted() {
    let $vm = this;
    mymap = L.map("map", { scrollWheelZoom: true }).setView(
      [35.038, 33.181],
      10
    );
    //this.mymap = mymap;
    L.tileLayer(
      //"http://tiles.mapc.org/basemap/{z}/{x}/{y}.png",
      "https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1Ijoia3Nib3JhIiwiYSI6ImNrMm9yeTliejEzMzkzY2xucHhlajI0bGQifQ.RIRPo0QfVPHZWDS7d-fTQA",
      {
        attribution:
          'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
        maxZoom: 18,
        id: "mapbox/streets-v11",
        accessToken:
          "pk.eyJ1Ijoia3Nib3JhIiwiYSI6ImNrMm9yeTliejEzMzkzY2xucHhlajI0bGQifQ.RIRPo0QfVPHZWDS7d-fTQA"
      }
    ).addTo(mymap);

    //let d1, d2, d3, d4, d5;
    //create devices with initial positions

    //console.log("device list: ", this.deviceList);
    mymap.on("contextmenu", this.createPolygon);

    let d1 = L.marker([35.038, 32.581])
      .addTo(mymap)
      .bindPopup("<b>Device 1</b>")
      .openPopup();

    $vm.deviceList.push({ name: "Device 1", status: "Active", handle: d1 });

    let d2 = L.marker([34.848, 32.681])
      .addTo(mymap)
      .bindPopup("<b>Device 2</b>")
      .openPopup();
    $vm.deviceList.push({ name: "Device 2", status: "Active", handle: d2 });

    let d3 = L.marker([34.958, 33.081])
      .addTo(mymap)
      .bindPopup("<b>Device 3</b>")
      .openPopup();
    $vm.deviceList.push({ name: "Device 3", status: "Active", handle: d3 });

    let d4 = L.marker([35.068, 33.481])
      .addTo(mymap)
      .bindPopup("<b>Device 4</b>")
      .openPopup();
    $vm.deviceList.push({ name: "Device 4", status: "Active", handle: d4 });

    let d5 = L.marker([35.078, 33.581])
      .addTo(mymap)
      .bindPopup("<b>Device 5</b>")
      .openPopup();

    $vm.deviceList.push({ name: "Device 5", status: "Active", handle: d5 });

    client.on("disconnect", function() {
      document.getElementById("server").classList.remove("active");
      document.getElementById("info").style.opacity = 0;
    });

    client.on("connect", function() {
      document.getElementById("server").classList.add("active");
      document.getElementById("info").style.opacity = 1;
    });
    client.on("areas", function(areas) {
      //console.log("areas message: ", areas);
      //console.log("local areas: ", $vm.rect);

      if ($vm.rect.length > 0)
        $vm.rect.forEach(item => mymap.removeLayer(item));

      $vm.areas = [];
      $vm.rect = [];
      if (areas.length > 0) {
        areas.forEach(poly => {
          let rec = L.rectangle(
            [
              [
                poly.geometry.coordinates[0][0][0],
                poly.geometry.coordinates[0][0][1]
              ],
              [
                poly.geometry.coordinates[0][2][0],
                poly.geometry.coordinates[0][2][1]
              ]
            ],
            { color: "#ff7800", weight: 1 }
          )
            .addTo(mymap)
            .bindTooltip(poly.properties.name, {
              closeOnClick: false,
              autoClose: false
            })
            .openTooltip();
          $vm.rect.push(rec);
        });
      }

      $vm.areas = [...areas];
    });

    client.on("deviceUpdates", function(message) {
      //this.count++;
      //console.log(message[0].lat, message[0].long);
      //if (this.count / 2 == 0) return;

      let devices = message;

      //console.log("[0]: ", $vm.deviceList[0]);
      if ($vm.deviceList[0].status == "Active") {
        mymap.removeLayer(d1);
        d1 = L.marker([devices[0].lat, devices[0].long])
          .addTo(mymap)
          .bindPopup("<b>Device 1</b>", {
            closeOnClick: false,
            autoClose: false,
            className: "device1"
          })
          .openPopup();
      }
      //$vm.deviceList.push({ name: "Device 1", status: "Active", handle: d1 });
      if ($vm.deviceList[1].status == "Active") {
        mymap.removeLayer(d2);
        d2 = L.marker([devices[1].lat, devices[1].long])
          .addTo(mymap)
          .bindPopup("<b>Device 2</b>", {
            closeOnClick: false,
            autoClose: false,
            className: "device2"
          })
          .openPopup();
      }
      //$vm.deviceList.push({ name: "Device 2", status: "Active", handle: d2 });
      if ($vm.deviceList[2].status == "Active") {
        mymap.removeLayer(d3);
        d3 = L.marker([devices[2].lat, devices[2].long])
          .addTo(mymap)
          .bindPopup("<b>Device 3</b>", {
            closeOnClick: false,
            autoClose: false,
            className: "device3"
          })
          .openPopup();
      }
      //$vm.deviceList.push({ name: "Device 3", status: "Active", handle: d3 });
      if ($vm.deviceList[3].status == "Active") {
        mymap.removeLayer(d4);
        d4 = L.marker([devices[3].lat, devices[3].long])
          .addTo(mymap)
          .bindPopup("<b>Device 4</b>", {
            closeOnClick: false,
            autoClose: false,
            className: "device4"
          })
          .openPopup();
      }
      //$vm.deviceList.push({ name: "Device 4", status: "Active", handle: d4 });
      if ($vm.deviceList[4].status == "Active") {
        mymap.removeLayer(d5);
        d5 = L.marker([devices[4].lat, devices[4].long])
          .addTo(mymap)
          .bindPopup("<b>Device 5</b>", {
            closeOnClick: false,
            autoClose: false,
            className: "device5"
          })
          .openPopup();
      }
      //$vm.deviceList.push({ name: "Device 5", status: "Active", handle: d5 });
      /*  this.deviceList.push(d1);
      this.deviceList.push(d2);
      this.deviceList.push(d3);
      this.deviceList.push(d4);
      this.deviceList.push(d5); */
    });
    let info = document.querySelector(".menu");
    info.onmouseenter = () => {
      this.info = true;
    };

    info.onmouseleave = () => {
      this.info = false;
    };
  },
  created() {
    //client = ioClient.connect("http://localhost:3000");
  },
  methods: {
    toggleTracking(device) {
      if (device == "Device 1") console.log("Device 1 Toggle");
      this.deviceList.forEach(item => {
        if (item.name == device)
          item.status = item.status == "Active" ? "NotActive" : "Active";
      });
    },
    resetLocations() {
      client.emit("reset", {});
      document.getElementById("start").disabled = true;
      document.getElementById("start").style.background = "grey";
      //console.log("deviceList: ", this.deviceList);
    },
    startTracking() {
      client.emit("start", {});
      document.getElementById("start").disabled = true;
      document.getElementById("start").style.background = "grey";
      //console.log("deviceList: ", this.deviceList);
    },
    createPolygon(e) {
      //console.log("You right clicked the map at ", e.latlng);
      Swal.fire({
        title: "Enter area name",
        input: "text",
        inputAttributes: {
          autocapitalize: "off"
        },
        showCancelButton: true,
        confirmButtonText: "Create",
        showLoaderOnConfirm: true,
        preConfirm: name => {
          return fetch(
            `http://localhost:3001/polygon?name=${name}&lat=${e.latlng.lat}&long=${e.latlng.lng}`
          )
            .then(response => {
              if (!response.ok) {
                throw new Error(response.statusText);
              }
              return response.json();
            })
            .catch(error => {
              Swal.showValidationMessage(`Request failed: ${error}`);
            });
        },
        allowOutsideClick: () => !Swal.isLoading()
      }).then(result => {
        if (result.value) {
          Swal.fire("Area created!", "To be shown on next update", "success");
          /* L.rectangle(
            [
              [
                result.value.geometry.coordinates[0][0][0],
                result.value.geometry.coordinates[0][0][1]
              ],
              [
                result.value.geometry.coordinates[0][2][0],
                result.value.geometry.coordinates[0][2][1]
              ]
            ],
            { color: "#ff7800", weight: 1 }
          )
            .addTo(mymap)
            .bindTooltip(result.value.properties.name, {
              closeOnClick: false,
              autoClose: false
            })
            .openTooltip(); */
          //this.areas.push(result.value);
        }
      });
    }
  }
};
</script>

<style>
#map {
  --height: 850px;
  height: 100vh;
  --width: 1500px;
  width: 100%;
}

.header {
  position: absolute;
  width: 300px;
  height: 50px;
  margin-left: 37%;
  background: black;
  color: white;
  font-weight: bold;
  z-index: 700;
  padding: 0.4%;
  box-sizing: border-box;
  text-align: center;
  font-size: 1.5rem;
}

.status {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: grey;
  position: fixed;
  margin-left: 12%;
  --margin-top: 2%;
  top: 1.8%;
}

.active {
  background: lightgreen;
}

.info {
  position: absolute;
  margin-left: 39.5%;
  margin-top: 2.5%;
  z-index: 600;
  width: 200px;
  height: 20px;
  color: orange;
  font-size: 0.9rem;
  background: black;
  opacity: 0;
  padding: 2px;
  text-align: center;
  font-family: Arial, Helvetica, sans-serif;
}

.start-btn {
  position: absolute;
  margin-left: 200px;
  margin-top: 120px;
  border-radius: 10px;
  z-index: 600;
  width: 200px;
  height: 60px;
  background-color: blueviolet;
  color: white;
}
.reset-btn {
  position: absolute;
  margin-left: 200px;
  margin-top: 50px;
  border-radius: 10px;
  z-index: 600;
  width: 200px;
  height: 60px;
  background-color: #4d79ff;
  opacity: 90%;
  color: white;
}
.device1 {
  background-color: chartreuse;
  padding: 5px;
  border-radius: 10px;
}

.device2 {
  background-color: orangered;
  padding: 5px;
  border-radius: 10px;
}

.device3 {
  background-color: blueviolet;
  padding: 5px;
  border-radius: 10px;
}

.device4 {
  background-color: yellow;
  padding: 5px;
  border-radius: 10px;
}

.device5 {
  background-color: mediumvioletred;
  padding: 5px;
  border-radius: 10px;
}

.menu {
  left: 73%;
  height: 20px;
  background: transparent;
  padding: 2%;
  color: white;
  font-size: 30px;
  top: 79%;
  position: absolute;
  z-index: 600;
}

.showinfo {
  height: 300px;
  position: absolute;
  width: 530px;
  background-color: white;
  left: 47%;
  top: 47%;
  padding: 1%;
  font-size: 18px;
  border-radius: 40px;
  border-bottom-right-radius: 1px;
  background: orange;
  z-index: 600;
  --border: 10px solid #1fa2ff;
}

.showinfo li {
  background: black;
  padding: 5px;
  margin: 2px;
  margin-bottom: 5px;
  color: white;
  border-radius: 10px;
  font-size: 0.9rem;
}

.showinfo p {
  margin-left: 40px;
  margin-bottom: 1px;
  padding-left: 4px;
  color: black;
  background: white;
  border: 1px solid white;
  border-radius: 5px;
  font-size: 0.8rem;
}
</style>
