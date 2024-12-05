<script setup lang="ts">
import { RouterLink, RouterView } from "vue-router";
import { ref, type Ref, version as vueVersion } from "vue";
import { Temporal } from "@js-temporal/polyfill";

const appName = __APP_NAME__;
const appVersion = __APP_VERSION__;

const currentYear: Ref<number> = ref(Temporal.Now.zonedDateTimeISO().year);
</script>

<template>
  <header>
    <nav>
      <RouterLink :to="{ name: 'Home' }">Home</RouterLink>
      <RouterLink :to="{ name: 'Documentation' }">Documentation</RouterLink>

      <RouterLink :to="{ name: 'About' }" class="to-the-right">About</RouterLink>
    </nav>
  </header>

  <main>
    <RouterView />
  </main>

  <footer>
    <div>
      <div>
        <em>Timestamp magician</em>
      </div>
      <div>
        App version: <var>{{ appVersion }}</var
        ><!-- ({{ appName }})-->
      </div>
      <div>
        Vue version: <var>{{ vueVersion }}</var>
      </div>
      <div>
        Copyright &copy; 2023-{{ currentYear }}
        <a href="https://anders.nemonisimors.com">Anders Gustafson</a>. All rights reserved.
      </div>
      <div>
        <a href="https://www.nemonisimors.com">
          <em>Nemo nisi mors</em>
        </a>
      </div>
    </div>

    <div>
      <a href="https://www.nemonisimors.com">
        <img src="@/assets/LogoNNM_blue_transparent_background.svg" alt="Logo NNM" />
      </a>
    </div>
  </footer>
</template>

<style scoped>
header {
  line-height: 1.5;
  max-height: 20vh;

  background-color: dodgerblue;

  position: sticky;
  top: 0;

  & nav {
    min-height: 3rem;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;

    font-weight: bold;

    & a {
      color: rgba(250, 250, 250, 1);
      border-radius: 1rem;
      margin: 0.5rem;
      text-decoration: none;

      display: inline-block;
      padding: 0 1rem;

      &.router-link-exact-active {
        background-color: white;
        color: dodgerblue;

        &:hover {
          cursor: default;
        }
      }

      &:hover {
        color: dodgerblue;
        background-color: white;
      }
    }

    & .to-the-right {
      margin-left: auto;
    }
  }
}

main {
  margin-bottom: 4rem;
}

footer {
  margin-block-start: auto;
  margin-block-end: 1rem;
  border-top: 1px solid var(--color-footer);
  font-size: 0.8rem;
  color: var(--color-footer);

  display: flex;
  flex-direction: row;
  justify-content: space-between;

  & img {
    max-width: min(20rem, 25vw);
  }

  & a {
    color: var(--color-footer);

    &:hover {
      color: blue;
    }
  }
}

@media screen and (max-width: 600px) {
  footer {
    flex-direction: column;
  }
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

/*
nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
}
*/
</style>
