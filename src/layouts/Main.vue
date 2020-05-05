<template>
  <q-layout>
    <q-layout-header reveal>
      <q-toolbar color="dark">
        <q-btn flat dense to="/">
          <img src="http://localhost:8000/img/logo2.png" alt="LOGO">
          <q-toolbar-title>
            Zóna Administrátorov
          </q-toolbar-title>
        </q-btn>
        <q-btn flat dense @click="$router.push('/products/create')" >
          Pridať produkt
        </q-btn>
      </q-toolbar>
    </q-layout-header>

    <q-page-container>
      <!-- This is where pages get injected -->
      <router-view />
      <q-ajax-bar color="primary"/>
    </q-page-container>

  </q-layout>
</template>
