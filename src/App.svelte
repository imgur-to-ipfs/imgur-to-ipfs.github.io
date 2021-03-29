<script>
	export let name;
	import Gallery from './Gallery.svelte';
	import queryString from "query-string";
	import {selectedImagesStore} from './stores.js'
	//import imgur from "imgur";
import { text } from 'svelte/internal';

	export let selectedImages = [];

	let accessToken = null;

	selectedImagesStore.subscribe(value => {
		selectedImages = value;
	})
	selectedImages = [];

	//export let selectedImages = [];

	function getThumbnailURL(imageURL) {
		let thumbnailBase = imageURL.split('.').slice(0, -1).join('.')
		let thumbnailExtension = imageURL.split('.')[-1];
		// "l" means "large thumbnail"
		return thumbnailBase + "l" + thumbnailExtension;
	}
	function removeItemOnce(arr, value) {
		var index = arr.indexOf(value);
		if (index > -1) {
			arr.splice(index, 1);
		}
		return arr;
	}

	function upload() {

	}

	function updateCounter() {
		let allImages = getAllImages();

		let imageCountInfo = document.getElementById("imageCountInfo");
		imageCountInfo.textContent = `${selectedImages.length} ${selectedImages.length == 1 ? "image" : "images"} selected.`;
		let selectAllButton = document.getElementById("selectAllButton");
		selectAllButton.textContent = selectedImages.length == allImages.length ? "Deselect all" : "Select all";

		let uploadButton = document.getElementById("uploadButton");
		uploadButton.style.display = selectedImages.length > 0 ? "inline-block" : "none";

	}

	function selectImage(image) {
		if (!selectedImages.includes(image)) {
			selectedImages.push(image);
			image.style.opacity = "100%";
			updateCounter();
		}
	}

	function deselectImage(image) {
		if (selectedImages.includes(image)) {
			removeItemOnce(selectedImages, image);
			image.style.opacity = "50%";
			updateCounter();
		}
	}
	
	function toggleImage(image) {
		if (selectedImages.includes(image)) {
			deselectImage(image);
		}
		else {
			selectImage(image);
		}
	}

	export function imageClicked(event) {
		let target = event.target;
		toggleImage(target);
	}

	function getAllImages() {
		return document.getElementsByClassName("imgurImage");
	}

	function selectAll() {
		let allImages = getAllImages();
		for (let i = 0; i < allImages.length; i++) {
			selectImage(allImages[i]);
		}
	}
	function deselectAll() {
		let allImages = getAllImages();
		for (let i = 0; i < allImages.length; i++) {
			deselectImage(allImages[i]);
		}
	}

	function toggleAll(event) {
		if (selectedImages.length < getAllImages().length) {
			selectAll();
		}
		else {
			deselectAll();
		}
		console.log(selectedImages);
	}

	function getImages(imageURLs) {
		let imageContent = "";
		for (let i = 0; i < imageURLs.length; i++) {
			imageContent += `<img src=\"${imageURLs[i]}\" onclick=\"imageClicked\" style=\"opacity:50%\">\n`;
		}

		console.log(imageContent);
		return imageContent;
	}

	export let parsed = {};

	let clientID = "a468c3a3e36e258";
	export let imgurUrl = `https://api.imgur.com/oauth2/authorize?client_id=${clientID}&response_type=token`;

	if (typeof window !== 'undefined') {
		console.log(window.location.href);
		parsed = window.location.hash;
		parsed = window.location.hash.replace("#", "?");
		console.log(parsed);
	parsed = queryString.parse(parsed);
	}

	console.log(parsed.access_token);
	/*if (process.browser) {
    const myCookieValue = getCookie("myCookie");
	}*/
	console.log(new Headers({
		'Authorization': `Bearer ${parsed.access_token}`,
		referrer: ''
	}).values().next())
	fetch('https://api.imgur.com/3/account/me/images', { 
		method: 'get', 
		headers: new Headers({
			'Authorization': `Bearer ${parsed.access_token}`
			//"referrer" : "api.imgur.com",
			/*"Access-Control-Allow-Origin": "*",
			"Access-Control-Allow-Credentials": "true",
			"Access-Control-Allow-Methods": "OPTIONS, GET, POST",
			"Access-Control-Allow-Headers": "Content-Type, Depth, User-Agent, X-File-Size, X-Requested-With, If-Modified-Since, X-File-Name, Cache-Control"
*/
			//'Content-Type': 'application/x-www-form-urlencoded'
		})
	}).then(function(res) {
    return res.json();
   })
  .then(function(resJson) {
	  console.log(resJson);
    return resJson;
   })

  console.log(parsed);
</script>

<main>
	<h1>Hello {name}!</h1>
	{#if parsed.access_token != undefined}
		<p id="imageCountInfo">0 images selected.</p>
		<button id="selectAllButton" on:click={toggleAll}>Select all</button>
		
		<div id="uploadTool">
			<p>Upload to:</p>
			<div>
				<input type="radio" id="localhost" name="uploadTarget" value="localhost" checked>
				<label for="localhost">localhost:</label>
				<input type="text" id="localhostPort" value="8080">
			</div>
			<div>
				<input type="radio" id="api" name="uploadTarget" value="api">
				<label for="api">Custom URL:</label>
				<input type="text" id="apiUrl">
			</div>
			<button id="uploadButton" on:click={upload} style="display:none">Upload</button>
			
		</div>
		<Gallery gap="10" onclickEvent={imageClicked} imageClass="imgurImage">
			{@html getImages(["https://via.placeholder.com/180x200/1", "https://via.placeholder.com/210x200/1", "https://via.placeholder.com/250x200/1",
		"https://via.placeholder.com/180x200/1", "https://via.placeholder.com/210x200/1", "https://via.placeholder.com/250x200/1",
		"https://via.placeholder.com/180x200/1", "https://via.placeholder.com/210x200/1", "https://via.placeholder.com/250x200/1"])}
		</Gallery>
	{:else}
		<a href="{imgurUrl}" rel="noreferrer">Login with Imgur</a>
	{/if}
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
	:global(img) { opacity: .9; transition: all .2s }
	:global(img):hover { opacity: 1; transform: scale(1.04) }

</style>