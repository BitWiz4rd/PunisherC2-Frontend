  <script setup>
  import { ref, provide } from 'vue';
  import { RouterLink, RouterView } from 'vue-router'
  import HelloWorld from './components/HelloWorld.vue'

  const C2_SERVER = "http://192.168.1.32:9090";
  provide('C2_SERVER', C2_SERVER);
  
  const API_KEY = "DQPIdjwqpodwoqidjio"
  provide('API_KEY', API_KEY);
  </script>

<style>
#app {
  width: 100%;
}
header {

}
</style>

  <template>
    <header>
      <img alt="Vue logo" class="logo" src="@/assets/logo.png" width="125" height="125" />
      <div class="wrapper">
        <nav>
          <RouterLink to="/">Home</RouterLink>
          <RouterLink to="/assets">Assets</RouterLink>
        </nav>
      </div>
    </header>
    <div class="blur header-blur"></div>

    <RouterView />
  </template>
