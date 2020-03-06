<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
        />

        <v-img
          alt="Vuetify Name"
          class="shrink mt-1 hidden-sm-and-down"
          contain
          min-width="100"
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-name-dark.png"
          width="100"
        />
      </div>

      <v-spacer></v-spacer>
      <router-link to='/' class="mr-2">
        <v-btn text>
          <span>Страница с таблицей</span>
          <v-icon>mdi-open-in-new</v-icon>
        </v-btn>
      </router-link>

      <router-link to="/about-me">
        <v-btn text>
          <span>Обо мне</span>
          <v-icon>mdi-open-in-new</v-icon>
        </v-btn>
      </router-link>

    </v-app-bar>

    <v-content>
      <router-view />
    </v-content>
  </v-app>
</template>
