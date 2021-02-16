<template>
  <v-card class="overflow-hidden" color="primary lighten-2" dark>
    <v-dialog v-model="alert" max-width="450">
      <v-card dark class="white--text">
        <h3 class="ma-5 pt-3">Are you sure you want to update the server?</h3>
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

    <v-card color="secondary" class="black--text" dark>
      <v-card-title
        ><span class="d-sm-none">LCT Public Spaces</span>
        <span class="d-sm-inline d-none"
          >Local Contact Tracing Public Spaces</span
        >
      </v-card-title>
      <v-card-subtitle>
        <v-row no-gutters
          ><v-col>Community: {{ nsp }}</v-col>
          <v-col class="text-center">
            <v-tooltip bottom>
              <template v-slot:activator="{ on, attrs }">
                <span v-bind="attrs" v-on="on" class="text-center">
                  <warnRoomCard />
                  <!-- update these props before using them in roomCard2
                      :visitor="selectedVisitor"
                    :log="log"
                    @warned="onWarned($event)"
                    @connect="onVisitorSelected()" -->
                </span>
              </template>
              <span>Warn Rooms</span>
            </v-tooltip></v-col
          >
          <v-col class="text-right">Visitor: {{ nickName }}</v-col></v-row
        >
      </v-card-subtitle>
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

      <v-tabs background-color="primary " color="secondary" center>
        <!-- Favorites  -->
        <v-tab>
          <v-icon left>
            mdi-heart
          </v-icon>
          <span class="d-sm-inline d-none">Favorites</span>
        </v-tab>

        <!-- Spaces  -->
        <v-tab>
          <v-icon left>
            mdi-home
          </v-icon>
          <span class="d-sm-inline d-none">Spaces</span>
        </v-tab>

        <!-- Spontaneous  -->
        <v-tab>
          <v-icon left>
            mdi-account-group
          </v-icon>

          <span class="d-sm-inline d-none">Gatherings</span>
        </v-tab>

        <!-- Favorites List -->
        <v-tab-item>
          <v-card tile>
            <v-card-text>
              <v-list dense shaped max-width="300">
                <v-subheader>Spaces you visited recently:</v-subheader>
                <v-divider></v-divider>
                <v-list-item-group v-model="favorite" color="primary">
                  <v-list-item v-for="(item, i) in favorites" :key="i">
                    <v-list-item-content>
                      <v-list-item-title v-text="item"></v-list-item-title>
                    </v-list-item-content>
                  </v-list-item>
                </v-list-item-group>
              </v-list>
            </v-card-text>
          </v-card>
        </v-tab-item>

        <!-- Spaces form -->
        <v-tab-item>
          <v-card>
            <v-card-text>
              Your local public spaces:
              <v-row>
                <v-col cols="auto">
                  <v-chip-group
                    v-model="selectedCategory"
                    mandatory
                    active-class="primary--text"
                  >
                    <v-chip filter>
                      <v-icon>mdi-store</v-icon>
                    </v-chip>
                    <v-chip filter>
                      <v-icon>mdi-silverware</v-icon>
                    </v-chip>

                    <v-chip filter>
                      <v-icon>mdi-bed</v-icon>
                    </v-chip>
                    <v-chip filter>
                      <v-icon>mdi-theater</v-icon>
                    </v-chip>
                  </v-chip-group>
                </v-col>
                <v-col cols="auto">
                  <v-autocomplete
                    v-model="selectedSpace"
                    :items="filteredSpaces"
                    :filter="customFilter"
                    color="white"
                    item-text="room"
                    label="Room"
                    clearable
                  ></v-autocomplete>
                </v-col>
              </v-row>
            </v-card-text>
          </v-card>
        </v-tab-item>

        <!-- Spontaneous Gatherings prompt-->
        <v-tab-item>
          <v-card>
            <v-card-text>
              <v-text-field
                v-model="selectedSpace"
                label="Identify the gathering"
                hint="Use a name others in the gathering would use"
                persistent-hint
                clearable
                autofocus
              ></v-text-field>
            </v-card-text>
            <v-card-actions>
              <v-btn class="mt-6" text color="error" @click="sheet = !sheet">
                Done
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-tab-item>
      </v-tabs>
    </v-card>
    <v-divider></v-divider>

    <v-snackbar v-model="hasSaved" :timeout="2000" absolute bottom left>
      You have entered
      {{ selectedSpace }}
    </v-snackbar>
  </v-card>
</template>

<script>
import warnRoomCard from "@/components/cards/warnRoomCard";

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
  components: {
    warnRoomCard,
  },
  computed: {
    selectedFavorite() {
      return this.favorites[this.favorite];
    },

    categories() {
      return [...new Set(data.map((v) => v.NAME))];
    },

    spaces() {
      return data.map((v) => {
        return { room: v.ID, id: v.CODE, category: v.NAME };
      });
    },
  },

  data() {
    return {
      selectedCategory: [],
      usePanels: false,
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
      this.alert = true;
    },

    saveMe() {
      this.alert = false;
      // this is where we send a Cypher query to RedisGraph
      const q = `MERGE (v:visitor{ name: '${this.nickName}'})
 MERGE (s:space{ name: '${this.selectedSpace}' })
 MERGE (v)-[r:visited{visitedOn:'${this.visitedOn}'}]->(s)`;
      this.log(q, "RedisGraph: add visit query");

      this.exposeEventPromise("logVisit", q).then((results) => {
        this.log(results, "ACK: logVisit");
        this.$emit("selectedSpace", { room: this.selectedSpace, id: "" });
        this.hasSaved = true;
      });
    },
  },

  watch: {
    favorite() {
      this.selectedSpace = this.selectedFavorite;
    },

    selectedCategory() {
      this.categorySelected = this.categories[this.selectedCategory];
      this.filteredSpaces = this.spaces.filter(
        (v) => v.category == this.categorySelected
      );
      this.selectedSpace = "";
    },
  },
  async mounted() {
    await Room.$fetch();
    this.panelState = [0]; // open only the 0th element of expansion-panels
  },
};
</script>
