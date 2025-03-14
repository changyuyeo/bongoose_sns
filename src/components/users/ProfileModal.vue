<template>
	<div class="modal fade" id="profileCardModal" tabindex="-1" aria-hidden="true">
		<div class="modal-dialog">
			<div class="modal-content">
				<div class="modal-header">
					<h5 class="modal-title">프로필 수정</h5>
				</div>
				<div class="modal-body">
					<form>
						<img
							:src="url || profileImage(userImageUrl)"
							:alt="user.name"
							class="profile-img"
							@click="onClickImageUpload"
						/>
						<div class="badge bg-danger profile-remove-img" @click="onclickImageClear">이미지 삭제</div>
						<input ref="imageInput" type="file" hidden @change="onChangeImages" />
						<div class="mb-3">
							<label class="col-form-label">닉네임</label>
							<input v-model="nickname" class="form-control" type="text" />
						</div>
						<div class="mb-3">
							<label class="col-form-label">1줄 자기소개</label>
							<textarea class="form-control" placeholder="1줄 자기소개를 입력해주세요!" v-model="description" />
						</div>
					</form>
				</div>
				<div class="modal-footer">
					<button type="button" class="btn btn-secondary" data-bs-dismiss="modal">취소하기</button>
					<button class="btn btn-primary" @click="onClickUpdateProfile" data-bs-dismiss="modal">수정하기</button>
				</div>
			</div>
		</div>
	</div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { updateUser } from '@/api/user'
import { deleteUserLocalStorage } from '@/utils/localStorage'
import { CommonMutationTypes } from '@/store/common/mutations'
import { AuthActionTypes } from '@/store/auth/actions'
import customAlert from '@/utils/customAlert'

export default defineComponent({
	props: {
		user: {
			type: Object,
			required: true
		}
	},

	data() {
		return {
			nickname: this.user.name,
			description: this.user.description,
			imageToggle: false,
			image: null as File | null,
			url: null as string | null,
			userImageUrl: (this.user.imageUrl || null) as string | null,
			noImageUpdate: false as boolean
		}
	},

	computed: {
		imageURI() {
			return process.env.VUE_APP_URI
		}
	},

	methods: {
		async onClickUpdateProfile() {
			this.$store.commit(`common/${CommonMutationTypes.START_LOADING}`)
			// 이미지수정 경우 분기처리
			if (this.image) {
				try {
					const formData = new FormData()
					formData.append('nickname', this.nickname)
					formData.append('description', this.description)
					formData.append('image', this.image)
					await updateUser(formData, 'form')
					// 유저 정보 갱신
					deleteUserLocalStorage()
					this.$store.dispatch(`auth/${AuthActionTypes.USER_INFO}`)
				} catch {
					customAlert('유저 정보 불러오기를 실패했습니다')
				} finally {
					this.$store.commit(`common/${CommonMutationTypes.END_LOADING}`)
				}
			} else {
				try {
					const userData = { nickname: this.nickname, description: this.description, defaultImage: this.noImageUpdate }
					await updateUser(userData, 'body')
					// 유저 정보 갱신
					deleteUserLocalStorage()
					this.$store.dispatch(`auth/${AuthActionTypes.USER_INFO}`)
					this.noImageUpdate = false
				} catch {
					customAlert('유저 정보 불러오기를 실패했습니다')
				} finally {
					this.$store.commit(`common/${CommonMutationTypes.END_LOADING}`)
				}
			}
		},
		// 이미지 추출
		profileImage(image: string | null): string {
			return image === null ? require('@/assets/images/default.png') : `${this.imageURI}/${image}`
		},
		onClickImageUpload() {
			if (this.imageToggle) {
				const refImage = this.$refs.imageInput as HTMLInputElement
				refImage.click()
				this.imageToggle = !this.imageToggle
			} else {
				this.url = null
				this.imageToggle = !this.imageToggle
			}
		},
		// 이미지 업로드
		onChangeImages(event: Event) {
			// 이미지 썸네일
			const target = event.target as HTMLInputElement
			const file: File = (target.files as FileList)[0]
			this.image = file
			this.url = URL.createObjectURL(file)
		},
		// 이미지 초기화
		onclickImageClear() {
			this.image = null
			this.url = null
			this.userImageUrl = null
			this.profileImage(null)
			this.noImageUpdate = true
		}
	}
})
</script>

<style lang="scss" scoped>
.profile-img {
	border-radius: 50%;
	width: 200px;
	height: 200px;
	display: block;
	margin: 15px auto;
	cursor: pointer;
	transition: 0.5s;
	&:hover {
		-webkit-filter: brightness(0.3);
		filter: brightness(0.3);
	}
}

.profile-remove-img {
	cursor: pointer;
	display: block;
	width: 85px;
	margin-left: auto;
}

button {
	margin: 0;
}
</style>
