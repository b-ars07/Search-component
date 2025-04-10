<template>
    <div class="app">
      <Search />
    </div>
  </template>
  
  <script setup>
  import Search from './components/Search.vue';
  </script>
  
  <style>
  body {
    margin: 0;
    padding: 0;
    font-family: Arial, sans-serif;
    background-color: #f7f7f7;
  }
  
  .app {
    max-width: 800px;
    margin: 50px auto;
    padding: 20px;
  }
  
  @media (max-width: 768px) {
    .app {
      margin: 20px auto;
      padding: 15px;
    }
  }
  </style>