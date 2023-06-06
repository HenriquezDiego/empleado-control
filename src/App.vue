<template>
  <v-app>
    <v-navigation-drawer
      :clipped="$vuetify.breakpoint.lgAndUp"
      width="300"
      fixed
      app
    >
      <v-list dense>
        <!-- INICIO -->
        <v-list-item to="/" exact color="primary">
          <v-list-item-action>
            <v-icon style="width: 25px">fa-solid fa-house</v-icon>
          </v-list-item-action>
          <v-list-item-title class="sidebarTitle">Inicio</v-list-item-title>
        </v-list-item>
        <v-list-item to="/empleados" exact color="primary">
          <v-list-item-action>
            <v-icon style="width: 25px">fa-sharp fa-solid fa-sitemap</v-icon>
          </v-list-item-action>
          <v-list-item-title class="sidebarTitle">Empleados</v-list-item-title>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <router-view />
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: "App",

  data: () => ({
    //
  }),
};
</script>
