<template>
  <b-navbar toggleable="lg" type="dark" variant="dark">
    <b-navbar-brand id="brand" tag="h1" class="mb-0" to="/">
      shepard frontend
    </b-navbar-brand>
    <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>
    <b-collapse id="nav-collapse" is-nav>
      <b-navbar-nav id="nav">
        <b-nav-item to="/explore">Collections</b-nav-item>
        <b-nav-item-dropdown text="Containers">
          <b-dropdown-item to="/files">Files</b-dropdown-item>
          <b-dropdown-item to="/structureddata">StructuredData</b-dropdown-item>
          <b-dropdown-item to="/timeseries">Timeseries</b-dropdown-item>
        </b-nav-item-dropdown>
        <b-nav-item to="/user">User</b-nav-item>
        <b-nav-item to="/about">About</b-nav-item>
      </b-navbar-nav>
      <b-navbar-nav class="ml-auto">
        <b-nav-item v-if="oidcIsAuthenticated" to="/about-user">
          Signed in as {{ fullName }}
        </b-nav-item>
      </b-navbar-nav>
    </b-collapse>
  </b-navbar>
</template>

<script lang="ts">
import Vue from "vue";
import { mapGetters } from "vuex";

export default Vue.extend({
  computed: {
    ...mapGetters("oidcStore", ["oidcIsAuthenticated", "oidcUser"]),
    hasAccess(): boolean {
      return this.oidcIsAuthenticated || this.$route.meta?.isPublic;
    },
    fullName(): string {
      return this.oidcUser?.family_name + ", " + this.oidcUser?.given_name;
    },
  },
});
</script>

<style scoped>
#brand {
  margin-left: 20px;
}
</style>
