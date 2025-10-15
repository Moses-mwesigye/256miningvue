<script setup>
import { ref } from 'vue';

const showModal = ref(false);
const uploadedPhoto = ref(null);
const loading = ref(false);
const errorMsg = ref('');
const modalPhone = ref('');
const phoneMatch = ref(false);

// Top right form state
const topDistrict = ref('');
const topEmail = ref('');
const topPhone = ref('');
const topSuccess = ref(false);

const IMAGGA_API_KEY = 'acc_b307138650d17ce';
const IMAGGA_API_SECRET = '8db152c996a390dd549f20d5b8059a62';

function handleTopFormSubmit() {
		if (!topDistrict.value || !topEmail.value || !topPhone.value) return;
		localStorage.setItem('email', topEmail.value);
		localStorage.setItem('phone', topPhone.value);
		topSuccess.value = true;
		// Clear fields after save
		topDistrict.value = '';
		topEmail.value = '';
		topPhone.value = '';
		setTimeout(() => { topSuccess.value = false; }, 1800);
}

function openModal() {
	showModal.value = true;
}
function closeModal() {
	showModal.value = false;
	uploadedPhoto.value = null;
	errorMsg.value = '';
	loading.value = false;
}
function handlePhotoUpload(event) {
	const file = event.target.files[0];
	uploadedPhoto.value = file;
	}

	function checkModalPhone() {
		const savedPhone = localStorage.getItem('phone');
		phoneMatch.value = (modalPhone.value && savedPhone && modalPhone.value === savedPhone);
}
async function submitPhoto() {
	if (!uploadedPhoto.value || !phoneMatch.value) return;
	loading.value = true;
	errorMsg.value = '';
	try {
		// Save image to localStorage under phone number
		const reader = new FileReader();
		reader.onload = async function(e) {
			const base64Image = e.target.result;
			const phone = modalPhone.value;
			// Save as array of images per phone
			let phoneImages = [];
			try {
				phoneImages = JSON.parse(localStorage.getItem('images_' + phone)) || [];
			} catch {}
			phoneImages.push(base64Image);
			localStorage.setItem('images_' + phone, JSON.stringify(phoneImages));

			// Step 1: Upload image to Imagga
			const formData = new FormData();
			formData.append('image', uploadedPhoto.value);
			const response = await fetch('https://api.imagga.com/v2/tags', {
				method: 'POST',
				headers: {
					'Authorization': 'Basic ' + btoa(IMAGGA_API_KEY + ':' + IMAGGA_API_SECRET)
				},
				body: formData
			});
			const data = await response.json();
			if (!data.result || !data.result.tags || data.result.tags.length === 0) {
				throw new Error('No tags found for this image.');
			}
			// Step 2: Use the top tag for Google search
			const topTag = data.result.tags[0].tag.en;
			const googleUrl = `https://www.google.com/search?q=${encodeURIComponent(topTag)}`;
			window.location.href = googleUrl;
			closeModal();
			loading.value = false;
		};
		reader.readAsDataURL(uploadedPhoto.value);
	} catch (err) {
		errorMsg.value = 'Image analysis failed. Please try again.';
		loading.value = false;
	}
}
</script>




<template>
		<div class="page-bg">
			<div class="form-card-wrapper">
				<form class="top-form" @submit.prevent="handleTopFormSubmit">
					<div class="top-form-vertical">
						<input class="top-input" v-model="topDistrict" required placeholder="District" />
						<input class="top-input" v-model="topEmail" type="email" required placeholder="Email" />
					</div>
					<input class="top-input" v-model="topPhone" type="tel" required placeholder="Phone" />
					<button class="top-btn" type="submit">Save</button>
				</form>
				<div v-if="topSuccess" class="top-success-popup">Successfully saved!</div>
			</div>
			<div class="glass-card">
			<p class="title">MINING MS</p>
			<p class="blue-text">Mineral Analysis</p>
			<button class="sample-btn" @click="openModal">Upload Photo for Lab Analysis</button>
		</div>
		<div v-if="showModal" class="modal-overlay">
			<div class="modal-card">
				<div class="modal-title">Upload Photo/sample data for Lab Analysis</div>
						<div style="display: flex; gap: 10px; align-items: flex-end;">
							<label class="modal-label" style="margin-bottom:0;">
								<span>Phone Number:</span>
								<input class="modal-input" type="tel" v-model="modalPhone" @input="checkModalPhone" placeholder="Enter phone" />
							</label>
							<label class="modal-label" style="margin-bottom:0;">
								<span>Select a photo:</span>
								<input class="modal-input" type="file" accept="image/*" @change="handlePhotoUpload" />
							</label>
						</div>
						<div v-if="modalPhone && !phoneMatch" style="color:red; font-size:0.98em; margin-top:4px;">Phone number does not match saved number.</div>
				<div v-if="uploadedPhoto" style="margin-top:10px; color:#2563eb; font-size:0.98em;">Selected: {{ uploadedPhoto.name }}</div>
						<div class="modal-actions">
							<button class="modal-btn cancel" @click="closeModal">Cancel</button>
							<button class="modal-btn" :disabled="!uploadedPhoto || loading || !phoneMatch" @click="submitPhoto">
								<span v-if="loading">Analyzing...</span>
								<span v-else>Submit</span>
							</button>
						</div>
						<div v-if="errorMsg" style="color:red; margin-top:10px;">{{ errorMsg }}</div>
				</div>
			</div>
		</div>
</template>

<!-- Modal and trigger button removed as per user request -->
<style scoped>
/* Top right form styles */
.form-card-wrapper {
	width: 100%;
	max-width: 420px;
	margin: 0 auto 18px auto;
	display: flex;
	flex-direction: column;
	align-items: stretch;
}
.top-form {
	display: flex;
	gap: 8px;
	align-items: flex-start;
	width: 100%;
	position: static;
	margin-bottom: 0;
	z-index: 1;
}
.top-form-vertical {
	display: flex;
	flex-direction: column;
	gap: 8px;
}
.top-input {
	padding: 6px 10px;
	border-radius: 6px;
	border: 1px solid #bdbdbd;
	font-size: 1em;
	min-width: 90px;
		width: 100%;
		box-sizing: border-box;
}

@media (max-width: 600px) {
	.top-form {
		position: static;
		flex-direction: column;
		align-items: stretch;
		width: 100%;
		padding: 0 12px;
	}
	.top-form-vertical {
		width: 100%;
	}
	.top-input {
		min-width: 0;
		margin-bottom: 8px;
		font-size: 1.05em;
	}
	.top-btn {
		width: 100%;
		margin-top: 8px;
	}
}
.top-btn {
	background: linear-gradient(90deg, #7c3aed 40%, #2563eb 100%);
	color: #fff;
	font-weight: 600;
	border: none;
	border-radius: 6px;
	padding: 6px 18px;
	font-size: 1em;
	cursor: pointer;
	transition: background 0.2s;
}
.top-btn:hover {
	background: linear-gradient(90deg, #2563eb 40%, #7c3aed 100%);
}
.top-success-popup {
	background: #e6ffe6;
	color: #2563eb;
	border: 1px solid #bdbdbd;
	border-radius: 8px;
	padding: 8px 18px;
	font-weight: 600;
	box-shadow: 0 2px 8px 0 rgba(37,99,235,0.10);
	margin-top: 8px;
	margin-bottom: 0;
	text-align: center;
	width: 100%;
	z-index: 1;
}
body {
	min-height: 100vh;
	margin: 0;
	font-family: 'Segoe UI', 'Roboto', Arial, sans-serif;
}
.page-bg {
	min-height: 100vh;
	width: 100vw;
	background: linear-gradient(135deg, #a18cd1 0%, #7c3aed 100%), url('https://www.transparenttextures.com/patterns/diamond-upholstery.png');
	background-blend-mode: overlay;
	display: flex;
	justify-content: center;
	align-items: center;
	padding: 0;
}

.glass-card {
	background: rgba(255, 255, 255, 0.22);
	border-radius: 28px;
	box-shadow: 0 8px 40px 0 rgba(124, 58, 237, 0.18), 0 2px 8px 0 rgba(31, 38, 135, 0.10);
	backdrop-filter: blur(18px) saturate(180%);
	-webkit-backdrop-filter: blur(18px) saturate(180%);
	border: 2px solid rgba(255, 255, 255, 0.25);
	padding: 48px 36px 36px 36px;
	max-width: 420px;
	width: 100%;
	display: flex;
	flex-direction: column;
	align-items: center;
	gap: 10px;
}

.title {
	font-size: 2.2em;
	font-weight: 700;
	margin-bottom: 18px;
	color: #222;
	letter-spacing: 1px;
	text-shadow: 0 2px 8px rgba(0,0,0,0.08);
}

.blue-text {
	color: #2563eb;
	font-weight: bold;
	font-size: 1.2em;
	margin-bottom: 24px;
	text-shadow: 0 1px 4px rgba(37,99,235,0.08);
}
.sample-btn {
	margin-top: 10px;
	padding: 14px 36px;
	background: linear-gradient(90deg, #7c3aed 40%, #2563eb 100%);
	color: #fff;
	font-size: 1.1em;
	font-weight: 600;
	border: none;
	border-radius: 14px;
	box-shadow: 0 2px 12px 0 rgba(124, 58, 237, 0.18);
	cursor: pointer;
	transition: background 0.2s, transform 0.2s;
}
.sample-btn:hover {
	background: linear-gradient(90deg, #2563eb 40%, #7c3aed 100%);
	transform: translateY(-2px) scale(1.04);
}

/* Modal Styles */
.modal-overlay {
	position: fixed;
	inset: 0;
	width: 100vw;
	height: 100vh;
	background: rgba(30, 16, 60, 0.25);
	display: flex;
	align-items: center;
	justify-content: center;
	z-index: 1000;
	pointer-events: auto;
	/* Fix for centering issues on some browsers */
	flex-direction: row;
}
.modal-card {
	background: rgba(255,255,255,0.32);
	border-radius: 22px;
	/* box-shadow removed for flat glassmorphism */
	backdrop-filter: blur(20px) saturate(180%);
	-webkit-backdrop-filter: blur(20px) saturate(180%);
	border: 2px solid rgba(124, 58, 237, 0.13);
	padding: 38px 32px 28px 32px;
	min-width: 340px;
	max-width: 95vw;
	width: 100%;
	display: flex;
	flex-direction: column;
	align-items: center;
	gap: 18px;
}
.modal-title {
	font-size: 1.4em;
	font-weight: 700;
	margin-bottom: 10px;
	color: #3a256e;
	text-align: center;
	letter-spacing: 0.02em;
}
.modal-label {
	display: flex;
	flex-direction: column;
	font-size: 1.08em;
	margin-bottom: 0;
	color: #5b3fa5;
	gap: 7px;
	font-weight: 600;
	letter-spacing: 0.02em;
}
.modal-label span {
	margin-bottom: 2px;
	color: #5b3fa5;
	font-size: 1em;
	font-weight: 600;
	letter-spacing: 0.01em;
	text-shadow: 0 1px 4px rgba(124, 58, 237, 0.08);
}
.modal-input {
	margin-top: 0;
	margin-bottom: 0;
	padding: 12px 16px;
	border-radius: 10px;
	border: 1.5px solid #bdbdbd;
	font-size: 1.08em;
	background: rgba(255,255,255,0.92);
	outline: none;
	transition: border 0.2s, box-shadow 0.2s;
	box-shadow: 0 1px 8px 0 rgba(124, 58, 237, 0.08);
}
.modal-input:focus {
	border: 1.5px solid #7c3aed;
	box-shadow: 0 2px 12px 0 rgba(124, 58, 237, 0.18);
}
.modal-actions {
	display: flex;
	justify-content: flex-end;
	gap: 16px;
	margin-top: 18px;
}
.modal-btn {
	background: linear-gradient(90deg, #7c3aed 40%, #2563eb 100%);
	color: #fff;
	font-weight: 700;
	border: none;
	border-radius: 10px;
	padding: 10px 28px;
	font-size: 1.08em;
	cursor: pointer;
	transition: background 0.2s, color 0.2s, box-shadow 0.2s;
	box-shadow: 0 2px 8px 0 rgba(124, 58, 237, 0.10);
}
.modal-btn.cancel {
	background: #f3f0fa;
	color: #5b3fa5;
	font-weight: 600;
	border: 1.5px solid #bdbdbd;
}
</style>
