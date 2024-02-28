<template>
  <b-navbar fixed-top spaced>
    <template #brand>
      <b-navbar-item tag="router-link" :to="{ path: '/' }">
        <b-icon icon="view-dashboard" size="is-medium"> </b-icon>
      </b-navbar-item>
    </template>
    <template #start>
      <b-navbar-item>
        <nuxt-link to="/info">Info</nuxt-link>
      </b-navbar-item>
    </template>

    <template #end>
      <b-navbar-item tag="div">
        <div class="buttons">
          <!--this needs to be updated to first have authentication not just Start App-->
          <nuxt-link to="/dashboard">
            <a class="button is-primary"> Start App </a>
          </nuxt-link>
        </div>
      </b-navbar-item>
    </template>
  </b-navbar>
</template>
