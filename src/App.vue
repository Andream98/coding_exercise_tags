<script setup>
	import Input from "./components/Input.vue";
</script>

<template>
	<div class="container">
		<div class="row align-items-center">
			<div class="col" id="inputContainer">
				<Input
					:pre-defined-tags="[
						'liver',
						'pain',
						'right',
						'left',
						'pancreas',
						'kidney',
						'brain',
						'severe_pain',
						'tumour',
						'cancer',
						'MRI',
						'CT',
						'male',
						'female',
						'bone',
						'shoulder',
						'hip',
						'XRAY',
						'knee',
						'spine',
						'head',
						'abdomen',
						'contrast',
						'fragment',
						'detached',
						'injury',
						'torn',
						'rotator',
						'cuff',
						'abdominal',
						'dilatation',
					]"
				/>
			</div>
		</div>
	</div>
</template>

<style>
	.row {
		min-height: 100vh;
	}
</style>
