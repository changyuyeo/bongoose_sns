<template>
	<!-- 이미지 개수에 따라 분기처리 -->
	<div
		v-if="images.length === 1"
		:style="{ 'background-image': `url(${imageURI}/${imageInfo[0]})` }"
		class="content-img"
	/>
	<!-- 2장 이상부터 이미지 캐러셀 적용 -->
	<ImagesCarousel v-else :images="imageInfo" />
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import ImagesCarousel from '@/components/posts/ImagesCarousel.vue'

export default defineComponent({
	components: {
		ImagesCarousel
	},

	props: {
		images: {
			type: Array,
			required: true
		}
	},
	computed: {
		imageURI() {
			return process.env.VUE_APP_URI
		},
		imageInfo() {
			// eslint-disable-next-line @typescript-eslint/no-explicit-any
			return this.images.map((v: any) => Object.values(v))
		}
	}
})
</script>

<style lang="scss" scoped>
.content-img {
	height: 500px;
	background-size: cover;
	background-position: center;
	border-radius: 10px;
	position: relative;
	margin: 0;
	@include media-breakpoint-down(lg) {
		height: 450px;
	}
	@include media-breakpoint-down(md) {
		height: 400px;
	}
}
</style>
