<template>
	<view class="container">
		<web-view :src="url"></web-view>
	</view>
</template>

<script>
	export default {
	  data() {
	    return {
	      url: "",
	    };
	  },
	  onLoad(e) {
		this.url = e.src
	  },
	  methods: {},
	};
</script>

<style lang="scss">
	
</style>

