<template>
  <v-card class="pa-4" flat height="550px">
    <v-row>
      <v-text-field
        hide-details
        prepend-icon="mdi-magnify"
        single-line
      ></v-text-field>

      <v-btn icon>
        <v-icon>mdi-crosshairs-gps</v-icon>
      </v-btn>

      <v-btn icon>
        <v-icon>mdi-dots-vertical</v-icon>
      </v-btn>
    </v-row>
    <v-row>
      <v-img
        alt="campus"
        class="shrink mr-2"
        contain
        src="@/assets/campus.jpg"
        height="450px"
      />
    </v-row>
  </v-card>
</template>

<script>
export default {};
</script>
