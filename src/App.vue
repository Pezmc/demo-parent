<script setup>
import { defineAsyncComponent } from "vue";

import HelloWorld from "./components/HelloWorld.vue";

function importExternal(url, libraryName, componentName) {
  const parts = url.split("/").reverse();

  if (parts.length === 0) return;

  const fileNameMatched = parts[0].match(/^(.*?)\.umd/);
  if (!fileNameMatched) return;
  const name = fileNameMatched[1];

  if (window[name]) return window[name];

  window[name] = new Promise((resolve, reject) => {
    const script = document.createElement("script");
    script.async = true;
    script.addEventListener("load", () => {
      resolve(window['MyLib']['HelloWorld']);
    });
    script.addEventListener("error", () => {
      reject(`Error loading ${url}`);
    });
    script.src = url;
    document.head.appendChild(script);
  });

  return window[name];
}

function importExternalComponent(url, libraryName, componentName = null) {
  return defineAsyncComponent(async () => {
    // Already loaded
    if (window[libraryName]?.[componentName]) {
      return window[libraryName][componentName]
    }

    // Mark component as loading by returning a promise that resolves with the module
    window[libraryName] = window[libraryName] || {}
    return window[libraryName][componentName] = (async () => {

      // Load the component library
      const externalImport = await import(url);

      if (!window[libraryName]) {
        throw new Error(`Loaded ${url} but library ${libraryName} not found, is that the correct name?`);
      }

      if (!window[libraryName][componentName]) {
        console.warn(`Failed to find ${componentName} in ${libraryName}`, window[libraryName])
        throw new Error(`Loaded ${url} and library ${libraryName}, but component ${componentName} didn't appear to be exported, is that the correct name?`);
      }

      // Library will register itself on window[libraryName]
      return window[libraryName][componentName]
    })();
  })
}

const AsyncComponent = importExternalComponent("http://localhost:4173/index.umd.js", 'MyLib', 'HelloWorld')
const AsyncComponent2 = importExternalComponent("http://localhost:4173/index.umd.js", 'MyLib', 'HelloWorld')
</script>

<template>
  <div>
    <a href="https://vitejs.dev" target="_blank">
      <img src="/vite.svg" class="logo" alt="Vite logo" />
    </a>
    <a href="https://vuejs.org/" target="_blank">
      <img src="./assets/vue.svg" class="logo vue" alt="Vue logo" />
    </a>
  </div>
  <HelloWorld msg="Sync Component" />
  <AsyncComponent msg="Async Component" />
  <AsyncComponent msg="This one is async too" />
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
