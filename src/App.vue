<template>
  <div id="app">
    <top-nav @clicked="showMenu"></top-nav>
    <div class="container-fluid">
      <div class="row">
        <left-nav v-show="isDisabled"></left-nav>
        <main-content></main-content>
      </div>
    </div>
  </div>
</template>

<script>
import MainContent from './components/MainContent';
import LeftNav from './components/LeftNav';
import TopNav from './components/TopNav';

export default {
	name: 'app',
	components: {
		MainContent,
		LeftNav,
		TopNav,
	},
	data() {
		return {
			isDisabled: false
		}
	},
	methods: {
		showMenu() {
			this.isDisabled = !this.isDisabled;
		}
	}
}
</script>

<style>
#app {
  font-family: -apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,"Helvetica Neue",Arial,"Noto Sans",sans-serif,"Apple Color Emoji","Segoe UI Emoji","Segoe UI Symbol","Noto Color Emoji";
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  font-size: .875rem;
}
</style>
