<template>
  <!-- Background Wrapper -->
  <div class="app-background"></div>
  <!-- This is the full-page background -->

  <!-- Gray Progress Strip -->
  <div class="progress-strip">
    <p>Still in progress...</p>
  </div>

  <RouterView />
</template>

<style scoped>
/* Background for the entire app */
.app-background {
  width: 100vw;
  height: 100vh;
  background: url('/bg1.jpg') no-repeat center center fixed;
  background-size: cover;
  position: fixed;
  top: 0;
  left: 0;
  z-index: -1; /* Pushes the background behind everything */
}

/* Gray Strip (Dimmed) */
.progress-strip {
  width: 100%;
  background-color: #b0b0b0; /* Dimmer gray */
  color: black;
  text-align: center;
  padding: 10px;
  font-size: 26px;
  font-weight: bold;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 1000;
  opacity: 0.85; /* Slight transparency */
  box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1); /* Soft shadow */
}
</style>
