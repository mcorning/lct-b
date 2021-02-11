<template>
  <v-card class="overflow-hidden" color="primary lighten-2" dark>
    <v-card-title>Local Contact Tracing Public Spaces</v-card-title>
    <v-card-subtitle
      >Community: {{ nsp }} Visitor: {{ nickName }}</v-card-subtitle
    >

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
      <v-expansion-panel>
        <v-expansion-panel-header color="primary lighten-3" dark>
          Your Local Spaces
        </v-expansion-panel-header>
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
            <v-btn
              :disabled="!canAddToFavorites"
              color="primary"
              dark
              @click="addToFavorites($event)"
            >
              Add to Favorites
            </v-btn>
          </v-card-text>
        </v-expansion-panel-content>
      </v-expansion-panel>
      <v-expansion-panel>
        <v-expansion-panel-header color="primary lighten-3" dark>
          Your Spontaneous Spaces
        </v-expansion-panel-header>
        <v-expansion-panel-content>
          <v-card-text>
            <v-card-subtitle
              >Examples include rallies, group hikes, and
              others</v-card-subtitle
            >
            <v-bottom-sheet v-model="sheet" inset>
              <template v-slot:activator="{ on, attrs }">
                <v-btn color="primary" dark v-bind="attrs" v-on="on">
                  Add one
                </v-btn>
              </template>
              {{ selectedAdHocGathering }}

              <v-sheet class="text-center" height="200px">
                <v-card-text>
                  <v-text-field
                    v-model="selectedAdHocGathering"
                    label="Identity"
                    hint="Use a name others in the gathering use"
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
                    close
                  </v-btn>
                </v-card-actions>
              </v-sheet>
            </v-bottom-sheet>
          </v-card-text>
        </v-expansion-panel-content>
      </v-expansion-panel>
    </v-expansion-panels>

    <!-- toolbar alternative -->
    <div v-if="false">
      <v-toolbar flat color="purple">
        <v-icon>mdi-account-group</v-icon>
        <v-spacer></v-spacer>

        <v-toolbar-title class="font-weight-light">
          Local Public Spaces in {{ nsp }}
        </v-toolbar-title>
      </v-toolbar>
      <v-card-text>
        <v-toolbar-title>Your Favorite Spaces</v-toolbar-title>
        <v-list flat dense>
          <v-list-item-group v-model="favorite" color="secondary">
            <v-list-item v-for="(item, i) in favorites" :key="i">
              <v-list-item-content>
                <v-list-item-title v-text="item"></v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </v-card-text>
      <v-card-text>
        <v-toolbar-title>Your Local Spaces</v-toolbar-title>
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
        <v-btn
          :disabled="!canAddToFavorites"
          color="primary"
          dark
          @click="addToFavorites($event)"
        >
          Add to Favorites
        </v-btn>
      </v-card-text>
      <v-card-text>
        <v-toolbar-title>Informal Gatherings</v-toolbar-title>
        <v-card-subtitle
          >Examples include rallies, group hikes, and others</v-card-subtitle
        >
        <v-bottom-sheet v-model="sheet" inset>
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark v-bind="attrs" v-on="on">
              Add one
            </v-btn>
          </template>
          {{ selectedAdHocGathering }}

          <v-sheet class="text-center" height="200px">
            <v-card-text>
              <v-toolbar-title>An Ad Hoc Gathering</v-toolbar-title>
              <v-text-field
                v-model="selectedAdHocGathering"
                label="Identity"
                hint="Use a name others in the gathering use"
                persistent-hint
                clearable
              ></v-text-field>
            </v-card-text>
            <v-card-actions>
              <v-btn class="mt-6" text color="error" @click="sheet = !sheet">
                close
              </v-btn>
            </v-card-actions>
          </v-sheet>
        </v-bottom-sheet>
      </v-card-text>
    </div>

    <v-divider></v-divider>
    <v-card-actions>
      <v-spacer></v-spacer>
      <v-btn
        :disabled="
          !selectedSpace && !selectedFavorite && !selectedAdHocGathering
        "
        color="primary"
        dark
        @click="save"
      >
        Log
        {{ selectedSpace || selectedAdHocGathering || selectedFavorite }}
      </v-btn>
    </v-card-actions>
    <v-snackbar v-model="hasSaved" :timeout="2000" absolute bottom left>
      You have entered
      {{ selectedSpace || selectedAdHocGathering || selectedFavorite }}
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
      default: "mpc",
    },
  },
  computed: {
    selectedFavorite() {
      let x = this.favorites[this.favorite];
      return x;
    },

    favorites() {
      const favs = Room.query()
        .orderBy("room")
        .get()
        .map((v) => v.room);
      return favs;
    },

    categories() {
      return data.map((v) => v.NAME);
    },

    spaces() {
      return data.map((v) => {
        return { room: v.ID, id: v.CODE, category: v.NAME };
      });
    },

    space() {
      return (
        this.selectedSpace ||
        this.selectedAdHocGathering ||
        this.selectedFavorite
      );
    },
  },

  data() {
    return {
      visitedOn: new Date().toLocaleDateString("en-US"),
      panelState: [],
      sheet: false,
      dialog: false,
      selectedAdHocGathering: "",
      favorite: -1,
      canAddToFavorites: false,
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
    addToFavorites() {
      Room.update(this.selectedSpace, null, this.nsp);
      this.canAddToFavorites = false;
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
      // this is where we send a Cypher query to RedisGraph
      const q = `MERGE (v:visitor{ name: '${this.nickName}'})
 MERGE (s:space{ name: '${this.space}' })
 MERGE (v)-[r:visited{visitedOn:'${this.visitedOn}'}]->(s)`;
      console.log(q);
      this.exposeEventPromise("addVisit", q).then((results) =>
        console.log("addVisit response:", results)
      );
    },
  },
  watch: {
    selectedSpace() {
      this.canAddToFavorites = this.selectedSpace;
    },
    selectedAdHocGathering() {
      this.hasSaved = true;
    },
  },
  async mounted() {
    await Room.$fetch();
    this.panelState = [0]; // open only the 0th element of expansion-panels
  },
};
</script>
