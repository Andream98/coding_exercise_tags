<script setup>
	defineProps({
		results: Array,
		shouldBeOpen: {
			type: Boolean,
			default: false,
		},
	});

	defineEmits(["emitTag"]);
</script>

<template>
	<div class="autocomplete">
		<ul class="autocomplete-results" v-show="shouldBeOpen">
			<li
				v-for="(result, i) in results"
				:key="i"
				@click="$emit('emitTag', result)"
				class="autocomplete-result"
			>
				{{ result }}
			</li>
		</ul>
	</div>
</template>

<style>
	.autocomplete {
		position: absolute;
	}

	.autocomplete-results {
		padding: 0;
		margin: 0;
		background-color: white;
		border: 1px solid #eeeeee;
		min-width: 250px;
		max-height: 6em;
		overflow: auto;
	}

	.autocomplete-result {
		list-style: none;
		text-align: left;
		padding: 4px 2px;
		cursor: pointer;
	}

	.autocomplete-result:hover {
		background-color: #4aae9b;
		color: white;
	}

	.autocomplete-result:active {
		background-color: #368072;
	}
</style>
