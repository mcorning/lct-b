<template>
  <v-card class="overflow-hidden" color="primary lighten-2" dark>
    <v-card-title>LCT Public Spaces</v-card-title>
    <v-card-subtitle
      >Community: {{ nsp }} Visitor: {{ nickName }}</v-card-subtitle
    >
    <v-dialog v-model="alert" max-width="450">
      <v-card dark class="white--text">
        <v-card-title>Are you sure you want to update the server?</v-card-title>
        <v-card-text class="white--text"
          >You cannot put this toothpaste back in the tube...</v-card-text
        >
        <v-card-actions>
          <v-btn color="red " text @click="saveMe">I'm sure</v-btn>
          <v-spacer></v-spacer>

          <v-btn color="green " text @click="alert = false">Never mind</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-card-actions>
      <v-spacer></v-spacer>
      <v-tooltip left>
        <template v-slot:activator="{ on, attrs }">
          <v-btn
            :disabled="!selectedSpace || !nickName"
            color="primary"
            dark
            v-bind="attrs"
            v-on="on"
            @click="save"
          >
            Log visit:
            {{ selectedSpace }}
          </v-btn>
        </template>
        <span>Send your visit to the server</span>
      </v-tooltip>
    </v-card-actions>

    <v-expansion-panels v-model="panelState" multiple popout dark>
      <v-expansion-panel>
        <v-expansion-panel-header color="primary lighten-3" dark>
          Your Favorite Spaces
        </v-expansion-panel-header>
        <v-expansion-panel-content>
          <v-list flat dense>
            <v-list-item-group v-model="favorite">
              <v-list-item v-for="(item, i) in favorites" :key="i">
                <v-list-item-content>
                  <v-list-item-title v-text="item"></v-list-item-title>
                </v-list-item-content>
              </v-list-item>
            </v-list-item-group>
          </v-list>
        </v-expansion-panel-content>
      </v-expansion-panel>

      <!-- Local Spaces -->
      <v-expansion-panel>
        <v-expansion-panel-header color="primary lighten-3" dark>
          Your Local Spaces
        </v-expansion-panel-header>

        <!-- Categories -->
        <v-expansion-panel-content>
          <v-card-text>
            <v-select
              v-model="categorySelected"
              :items="categoryLabels"
              item-text="label"
              item-value="NAME"
              return-object
              label="Category"
              hint="We group public spaces by these categories."
              persistent-hint
              @change="onChangeCategory"
            ></v-select>
            <v-autocomplete
              v-model="selectedSpace"
              :items="filteredSpaces"
              :filter="customFilter"
              color="white"
              item-text="room"
              label="Room"
              clearable
            ></v-autocomplete>
          </v-card-text>
        </v-expansion-panel-content>
      </v-expansion-panel>

      <!-- Ad Hoc visits -->
      <v-expansion-panel>
        <v-expansion-panel-header color="primary lighten-3" dark>
          Your Spontaneous Spaces
        </v-expansion-panel-header>
        <v-expansion-panel-content>
          <v-card-text>
            <v-card-subtitle
              >Spontaneous spaces are unstructured gatherings. Examples of
              deliberate public gatherings include rallies, raves, and group
              hikes. Examples of accidental gatherings are waiting for a bus,
              train, or plane. A common example includes sharing a ride in a
              vehicle.</v-card-subtitle
            >
            <v-bottom-sheet v-model="sheet" inset>
              <template v-slot:activator="{ on, attrs }">
                <v-btn color="primary" dark v-bind="attrs" v-on="on" block>
                  Add such a gathering...
                </v-btn>
              </template>

              <v-sheet class="text-center" height="200px">
                <v-card-text>
                  <v-text-field
                    v-model="selectedSpace"
                    label="Identify the gathering"
                    hint="Use a name others in the gathering would use"
                    persistent-hint
                    clearable
                  ></v-text-field>
                </v-card-text>
                <v-card-actions>
                  <v-btn
                    class="mt-6"
                    text
                    color="error"
                    @click="sheet = !sheet"
                  >
                    Done
                  </v-btn>
                  <v-btn class="mt-6" text color="error" @click="cancel">
                    Cancel
                  </v-btn>
                </v-card-actions>
              </v-sheet>
            </v-bottom-sheet>
          </v-card-text>
        </v-expansion-panel-content>
      </v-expansion-panel>
    </v-expansion-panels>

    <v-divider></v-divider>

    <v-snackbar v-model="hasSaved" :timeout="2000" absolute bottom left>
      You have entered
      {{ selectedSpace }}
    </v-snackbar>
  </v-card>
</template>

<script>
import Room from "@/models/Room";

import { data } from "@/assets/data/sistersBusiness.json";

export default {
  props: {
    nickName: {
      type: String,
      default: "",
    },
    favorites: {
      type: Array,
    },
    log: { type: Function },
  },
  computed: {
    selectedFavorite() {
      return this.favorites[this.favorite];
    },

    categories() {
      return data.map((v) => v.NAME);
    },

    spaces() {
      return data.map((v) => {
        return { room: v.ID, id: v.CODE, category: v.NAME };
      });
    },
  },

  data() {
    return {
      alert: false,
      visitedOn: new Date().toLocaleDateString("en-US"),
      panelState: [],
      sheet: false,
      dialog: false,
      favorite: -1,
      nsp: "sisters",
      categoryLabels: [
        { NAME: "RES", label: "Food and Drink" },
        { NAME: "RETAIL", label: "Retail" },
        { NAME: "LODG", label: "Lodging" },
        { NAME: "ENTER", label: "Entertainment" },
      ],
      filteredSpaces: [],
      categorySelected: "",
      selectedSpace: "",
      hasSaved: false,
      model: null,
    };
  },

  methods: {
    cancel() {
      this.sheet = !this.sheet;
      this.selectedSpace = "";
    },

    onChangeCategory() {
      this.filteredSpaces = this.spaces.filter(
        (v) => v.category == this.categorySelected.NAME
      );
    },

    customFilter(item, queryText) {
      const textOne = item.room.toLowerCase();
      const searchText = queryText.toLowerCase();
      const res = textOne.indexOf(searchText) > -1;
      return res;
    },

    exposeEventPromise(event, data) {
      let self = this;
      return new Promise(function(resolve) {
        self.$socket.emit(event, data, (results) => {
          resolve(results);
        });
      });
    },
    save() {
      this.hasSaved = true;
      this.alert = true;
    },

    saveMe() {
      this.alert = false;
      // this is where we send a Cypher query to RedisGraph
      const q = `MERGE (v:visitor{ name: '${this.nickName}'})
 MERGE (s:space{ name: '${this.selectedSpace}' })
 MERGE (v)-[r:visited{visitedOn:'${this.visitedOn}'}]->(s)`;
      console.log(q);
      this.exposeEventPromise("logVisit", q).then((results) => {
        this.log(results, "ACK: logVisit");
        this.$emit("selectedSpace", { room: this.selectedSpace, id: "" });
      });
    },
  },

  watch: {
    favorite() {
      this.selectedSpace = this.selectedFavorite;
    },
  },
  async mounted() {
    await Room.$fetch();
    this.panelState = [0]; // open only the 0th element of expansion-panels
  },
};
</script>
