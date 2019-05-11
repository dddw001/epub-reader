<template>
  <div id="app">
    <ebook></ebook>
  </div>
</template>

<script>
import ebook from './components/ebook';

export default {
  name: 'App',
  components: {
    ebook,
  },
};
document.addEventListener('DOMContentLoaded', () => {
  const html = document.querySelector('html');
  let fontSize = window.innerWidth / 10;
  fontSize = fontSize > 50 ? 50 : fontSize;
  html.style.fontSize = fontSize + 'px';
});
</script>

<style>
</style>
