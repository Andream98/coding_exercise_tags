<script setup>
	import { computed, onMounted, onUnmounted, ref } from "vue";
	import AutoComplete from "./AutoComplete.vue";
	import Cursor from "../utils/Cursor";

	const props = defineProps({
		preDefinedTags: {
			type: Array,
			required: false,
			default: () => [],
		},
	});

	const contentEditable = ref(null);
	const searchResults = ref([]);
	const latestMouseEventWasInsideTheInput = ref(false);
	const cursorIsPlacedAfterAnHashtag = ref(false);
	const cursorLatestPosition = ref(
		Cursor.getCurrentCursorPosition(contentEditable.value)
	);

	const autoCompleteDropDownShouldBeOpen = computed(() => {
		if (
			latestMouseEventWasInsideTheInput.value &&
			cursorIsPlacedAfterAnHashtag.value &&
			searchResults.value.length
		) {
			return true;
		}

		return false;
	});

	function updateCursorPosition() {
		cursorLatestPosition.value = Cursor.getCurrentCursorPosition(
			contentEditable.value
		);
	}

	function inputEventHandler() {
		latestMouseEventWasInsideTheInput.value = true;

		positionChangeHandler();

		filterResults();

		formatContent();
	}

	function formatContent() {
		let textContent = contentEditable.value.innerText,
			validPattern = /(?<![A-zÀ-ú0-9])(#[A-zÀ-ú0-9]{1,})(?![A-zÀ-ú0-9#])/gm;

		const matches = textContent.match(validPattern);

		let manipulatedTextContent = textContent;

		if (matches) {
			for (const match of matches) {
				let placeholder = null;
				placeholder = Object.assign(document.createElement("span"), {
					className: "tag",
				});

				placeholder.innerHTML = match;

				let matchNotWrappedInHTMLRegEx = new RegExp(
					`(?<![>])(${match})(?![<])`,
					"gm"
				);

				manipulatedTextContent = manipulatedTextContent.replace(
					matchNotWrappedInHTMLRegEx,
					placeholder.outerHTML
				);
			}
		}

		contentEditable.value.innerHTML = manipulatedTextContent;

		Cursor.setCurrentCursorPosition(
			cursorLatestPosition.value,
			contentEditable.value
		);

		contentEditable.value.focus();
	}

	function positionChangeHandler() {
		let currentWord = getCurrentWord(
			contentEditable.value.innerText,
			cursorLatestPosition.value
		);

		if (currentWord.length > 1 && currentWord.startsWith("#")) {
			cursorIsPlacedAfterAnHashtag.value = true;
		} else {
			cursorIsPlacedAfterAnHashtag.value = false;
		}
	}

	function getCurrentWord(inputContent, position, withHashtag = true) {
		let currentWord = "",
			currentPosition = position;

		while (
			typeof inputContent[currentPosition - 1] !== "undefined" &&
			inputContent[currentPosition - 1].match(/([A-zÀ-ú0-9#])/gm)
		) {
			currentWord = inputContent[currentPosition - 1] + currentWord;
			currentPosition -= 1;
		}

		currentPosition = position;

		if (
			typeof inputContent[currentPosition] !== "undefined" &&
			inputContent[currentPosition].match(/([A-zÀ-ú0-9])/gm)
		) {
			currentWord = currentWord + inputContent[currentPosition];

			while (
				typeof inputContent[currentPosition + 1] !== "undefined" &&
				inputContent[currentPosition + 1].match(/([A-zÀ-ú0-9#])/gm)
			) {
				currentWord = currentWord + inputContent[currentPosition + 1];
				currentPosition += 1;
			}
		}

		return withHashtag ? currentWord : currentWord.replace("#", "");
	}

	function filterResults() {
		let currentWord = getCurrentWord(
			contentEditable.value.innerText,
			cursorLatestPosition.value,
			false
		);

		searchResults.value = props.preDefinedTags.filter((tag) => {
			let anotherEqualTagExists = false;

			document
				.querySelectorAll('div[contenteditable="true"] span.tag')
				.forEach((span) => {
					if (span.innerText.replace("#", "") === tag) {
						anotherEqualTagExists = true;
					}
				});

			if (
				tag.toLowerCase().indexOf(currentWord.toLowerCase()) > -1 &&
				anotherEqualTagExists === false
			) {
				return true;
			}
		});
	}

	function addTag(tag) {
		contentEditable.value.innerText = addNewTag(tag);

		formatContent();

		filterResults();

		placeCursorAfterCurrentWord();
	}

	function addNewTag(tag) {
		return contentEditable.value.innerHTML.replace(
			`<span class="tag">${getCurrentWord(
				contentEditable.value.innerText,
				cursorLatestPosition.value
			)}</span>`,
			"#" + tag
		);
	}

	function placeCursorAfterCurrentWord() {
		let inputContent = contentEditable.value.innerText,
			currentPosition = cursorLatestPosition.value;

		while (
			typeof inputContent[currentPosition + 1] !== "undefined" &&
			inputContent[currentPosition + 1].match(/([A-zÀ-ú0-9#])/gm)
		) {
			currentPosition += 1;
		}

		currentPosition += 1;

		Cursor.setCurrentCursorPosition(currentPosition, contentEditable.value);
	}

	onMounted(() => {
		contentEditable.value = document.querySelector(
			'div[contenteditable="true"]'
		);
		
		document.addEventListener("click", handleClick);
	});
	onUnmounted(() => {
		document.removeEventListener("click", handleClick);
	});
	function handleClick(event) {
		if (document.querySelector("#inputContainer").contains(event.target)) {
			latestMouseEventWasInsideTheInput.value = true;
		} else {
			latestMouseEventWasInsideTheInput.value = false;
		}
	}
</script>

<template>
	<div
		contenteditable="true"
		class="form-control form-control-lg"
		@click="updateCursorPosition(), inputEventHandler()"
		@keyup="updateCursorPosition(), inputEventHandler()"
	></div>

	<AutoComplete
		:results="searchResults"
		:should-be-open="autoCompleteDropDownShouldBeOpen"
		@emit-tag="addTag"
	/>
</template>

<style>
	.tag {
		color: orange;
	}
</style>
