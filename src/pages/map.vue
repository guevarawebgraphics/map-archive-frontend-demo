<template>
  <div class="q-pa-md">
    <q-layout
      view="lHh Lpr lFf"
      container
      style="height: 900px"
      class="shadow-2 rounded-borders"
    >
      <q-header elevated>
        <q-toolbar>
          <q-avatar>
            <img src="https://cdn.quasar.dev/logo/svg/quasar-logo.svg" />
          </q-avatar>
          <q-toolbar-title>
            <strong>Location</strong> List Management
          </q-toolbar-title>
        </q-toolbar>
      </q-header>

      <div style="height: 350px;">
        <div
          class="info"
          style="height: 15%"
        >
          <span>Center: {{ center }}</span>
          <span>Zoom: {{ zoom }}</span>
          <span>Bounds: {{ bounds }}</span>
        </div>
        <l-map
          style="height: 80%; width: 100%"
          :zoom="zoom"
          :center="center"
          @update:zoom="zoomUpdated"
          @update:center="centerUpdated"
          @update:bounds="boundsUpdated"
        >
          <l-tile-layer :url="url"></l-tile-layer>
          <l-marker
            v-for="(loc, i) in locations"
            :key="i"
            :lat-lng="loc.coords"
          ></l-marker>
        </l-map>
      </div>

      <q-page-container>
        <q-page padding>
          <div class="row">
            <div class="col">
              <button @click="printPDF()">asd</button>
              <q-card
                class="my-card text-white"
                style="background: radial-gradient(circle, #35a2ff 0%, #014a88 100%)"
              >
                <q-card-section>
                  <div class="text-h6">Instructions</div>
                  <div class="text-subtitle2">by Administrator</div>
                </q-card-section>

                <q-card-section class="q-pt-none">
                  <ul
                    v-for="(instruction, index) in instructions"
                    :key="index"
                  >
                    <li>{{ instruction.content }}</li>
                  </ul>
                </q-card-section>
              </q-card>
            </div>
          </div>

          <div class="row">
            <div class="col">
              <q-card class="my-card">
                <q-card-section>
                  <q-input
                    outlined
                    label="Location / Address"
                    v-model="name"
                  />
                </q-card-section>

                <q-separator />

                <q-card-actions align="right">
                  <q-btn
                    color="red"
                    icon="delete_forever"
                    label="Delete"
                    @click="remove"
                  />
                  <q-btn
                    v-show="selectedLocation === null"
                    color="primary"
                    icon="note_add"
                    label="Add"
                    @click="create"
                  />

                  <q-btn
                    v-show="selectedLocation != null"
                    color="primary"
                    icon="save"
                    label="Edit"
                    @click="edit"
                  />
                </q-card-actions>
              </q-card>
            </div>
          </div>

          <list-locations
            :selectedLocation="selectedLocation"
            :locations="locations"
            @getLocation="selectLoc"
          />

          <!-- place QPageScroller at end of page -->
          <q-page-scroller
            expand
            position="top"
            :scroll-offset="150"
            :offset="[0, 0]"
          >
            <div class="col cursor-pointer q-pa-sm bg-blue-9 text-white text-center">
              Scroll back up...
            </div>
          </q-page-scroller>
        </q-page>
      </q-page-container>
    </q-layout>

    <!-- for modal -->
    <q-dialog
      v-model="promptPass"
      persistent
    >
      <q-card>
        <q-card-section class="row items-center">
          <q-avatar
            icon="warning"
            color="primary"
            text-color="white"
          />
          <p class="q-ml-md">Deleting {{ name }} from the list</p>
        </q-card-section>

        <q-card-section class="row items-center">
          <p>Input admin password for confirmation</p>
        </q-card-section>

        <q-card-section class="row items-center">
          <label><b>Password</b> </label>
          <q-input
            type="password"
            v-model="passwordInput"
            dense
          />
        </q-card-section>

        <q-card-actions align="right">
          <q-btn
            flat
            label="Cancel"
            color="primary"
            v-close-popup
            @click="cancelled"
          />
          <q-btn
            flat
            label="Confirm"
            color="primary"
            @click="removeLocation"
            v-close-popup
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<script>
import ListLocations from "src/components/ListLocations.vue";
import { latLng } from "leaflet";
import { LMap, LTileLayer } from "vue2-leaflet";

export default {
  components: {
    ListLocations,
    LMap,
    LTileLayer
  },
  data() {
    return {
      name: null,
      isExisting: null,
      selectedLocation: null,
      promptPass: false,
      locId: null,
      notSoSecretPassword: "adminpass",
      passwordInput: "",
      instructions: [
        {
          content: "Location input field must have a value"
        },
        {
          content: "You can't duplicate existing location"
        },
        {
          content:
            "you must provide an Administrator's password for you to delete a record"
        },
        {
          content: "Password is: adminpass"
        }
      ],
      locations: [
        {
          address: "Brgy. Osorio",
          coords: {}
        },
        {
          address: "Brgy. Gregorio",
          coords: {}
        },
        {
          address: "Brgy. Conchu",
          coords: {}
        },
        {
          address: "Brgy. Hugo Perez",
          coords: {}
        }
      ],
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      zoom: 10,
      center: [14.27853, 120.87475],
      bounds: null
    };
  },
  mounted() {
    this.locations = [];

    this.locationsService = this.$dbCon.wingsService("locations").init();

    this.locationsService.on("dataChange", locs => {
      this.locations = locs;
    });
  },
  methods: {
    zoomUpdated(zoom) {
      this.zoom = zoom;
    },
    centerUpdated(center) {
      this.center = center;
    },
    boundsUpdated(bounds) {
      this.bounds = bounds;
    },
    printPDF() {
      const docDefinition = {
        content: [
          "First paragraph",
          "Another paragraph, this time a little bit longer to make sure, this line will be divided into at least two lines"
        ]
      };
      this.$pdfMake.createPdf(docDefinition).open();
    },
    notify(type, message) {
      this.$q.notify({
        type: `${type}`,
        message: `${message}`
      });
    },
    async create() {
      const result = await this.$axios.get(
        `http://geocode.xyz/${this.name}?json=1`
      );
      console.log("address", result.data);
      if (this.name === null) {
        // checks if input is blank
        this.notify(`negative`, `Location / address field must have a value`);
        this.name = null;
        this.selectedLocation = null;
      } else {
        // for checking
        this.isExisting = this.locations.some(
          element => element.address === this.name
        );
        if (this.isExisting) {
          // check if input exists in locations array
          this.notify(`warning`, `${this.name} already exists`);
          this.name = null;
          this.selectedLocation = null;
        } else {
          // add to list
          // this.locations.push({
          //   address: this.name,
          //   coords: latLng(latt, longt)
          // });

          const { latt, longt } = result.data;
          this.locationsService.create({
            address: this.name,
            coords: latLng(latt, longt)
          });
          this.notify(`positive`, `${this.name} has been added to the list`);
          this.name = null;
          this.selectedLocation = null;
        }
      }
    },
    selectLoc(location) {
      // select location
      this.selectedLocation = location;
      this.name = this.selectedLocation.address;
      this.locId = this.selectedLocation._id;
    },
    removeLocation() {
      if (this.passwordInput != this.notSoSecretPassword) {
        //check for password
        this.notify(`negative`, `Incorrect password`);
      } else {
        // delete from array
        this.notify(`info`, `Location has been removed from the list`);
        this.locationsService.remove(
          this.locId
        );
        this.selectedLocation = null;
        this.name = null;
        this.passwordInput = "";
      }
    },
    cancelled() {
      // cancel deletion
      this.notify(`info`, `Deletion of '${this.name}' has been cancelled`);
      this.name = null;
      this.selectedLocation = null;
    },
    remove() {
      if (this.selectedLocation === null) {
        // check if location is selected
        this.notify(`negative`, `Select a location item first`);
      } else {
        this.promptPass = true;
      }
    },
    async edit() {
      // for checking
      this.isExisting = this.locations.some(
        element => element.address === this.name
      );

      if (this.isExisting) {
        // check if input exists in locations array
        this.notify(`warning`, `${this.name} already exists`);
        this.name = null;
        this.selectedLocation = null;
      } else {
        if (this.name === null) {
          // checks if input is empty
          this.notify(`warning`, `Location / address field must have a value`);
          this.name = null;
          this.selectedLocation = null;
        } else {
          // update location from array
          this.notify(`info`, `Location '${this.name}' has been updated`);
          const result = await this.$axios.get(
            `http://geocode.xyz/${this.name}?json=1`
          );
          console.log(result);
          const { latt, longt } = result.data;
          this.locationsService.update(
            this.locId,
            { address: this.name, coords: latLng(latt, longt) },
          );

          this.name = null;
          this.selectedLocation = null;
        }
      }
    }
  }
};
</script>

<style>
.separator {
  margin: 10px;
}
</style>
