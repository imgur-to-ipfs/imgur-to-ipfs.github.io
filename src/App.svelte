<svelte:head>
	<link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.8.2/css/all.css">
	<!-- Google Fonts -->
	<!--
		<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap">
	-->
	
	<!-- Bootstrap core CSS -->
	<!--<link href="bootstrap.min.css" rel="style" />-->
	<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-eOJMYsd53ii+scO/bJGFsiCZc+5NDVN2yr8+0RDqr0Ql0h+rP48ckxlpbzKgwra6" crossorigin="anonymous">
	<!--<link href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.4.1/css/bootstrap.min.css" rel="stylesheet">-->
	<!-- Material Design Bootstrap -->
	<!--
	<link href="https://cdnjs.cloudflare.com/ajax/libs/mdbootstrap/4.16.0/css/mdb.min.css" rel="stylesheet">
	-->
	
	<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700|Material+Icons">
<link rel="stylesheet" href="https://unpkg.com/bootstrap-material-design@4.1.1/dist/css/bootstrap-material-design.min.css" integrity="sha384-wXznGJNEXNG1NFsbm0ugrLFMQPWswR3lds2VeinahP8N0zJw9VWSopbjv2x7WCvX" crossorigin="anonymous">
</svelte:head>

<script>
	export let name;
	import Gallery from './Gallery.svelte';
	import queryString from "query-string";
	import {selectedImagesStore} from './stores.js'
	import {getRoot} from './ipfs.js'
	//import ipfshttpclient from '@rollup/ipfshttpclient';
	//import imgur from "imgur";
import { text } from 'svelte/internal';

	export let selectedImages = [];
	export let imageURLs = [];

	/*console.log(getRoot);
	console.log(window.Ipfs);
	let Ipfs = window.Ipfs;
	console.log("A");
	const node = Ipfs.create();
	console.log("B");
	console.log(node);*/
	//node.version().then(value => console.log("Value: " + value));
	//Ipfs.create().then(node => node.version()).then(value => console.log("Value " + value));
	console.log("C");
	

	let accessToken = null;

	selectedImagesStore.subscribe(value => {
		selectedImages = value;
	})
	selectedImages = [];

	//export let selectedImages = [];

	function getThumbnailURL(imageURL) {
		let thumbnailBase = imageURL.split('.').slice(0, -1).join('.');
		let thumbnailExtension = imageURL.split('.').slice(-1)[0];
		// "l" means "large thumbnail"
		return thumbnailBase + "m." + thumbnailExtension;
	}
	function getOriginalURL(thumbnailURL) {
		let thumbnailBase = thumbnailURL.split('.').slice(0, -1).join('.');
		thumbnailBase = thumbnailBase.slice(0, -1)
		let thumbnailExtension = thumbnailURL.split('.').slice(-1)[0];
		return thumbnailBase + "." + thumbnailExtension;

	}
	function removeItemOnce(arr, value) {
		var index = arr.indexOf(value);
		if (index > -1) {
			arr.splice(index, 1);
		}
		return arr;
	}

	function uploadOnIPFS(urls) {
		Ipfs.create().then(async function (node) {
			let list = document.getElementById("cidList");

			let files = [];
			console.log(urls);
			for (let i = 0; i < urls.length; i++) {
				let image = await fetch(urls[i],
				{
					//"mode" : "no-cors"
				});
				let content = {
					content: (await image.blob())
				};
				files.push(content);
			}

			for (let i = 0; i < files.length; i++) {
				let result = (await node.add(files[i])).path;
				let element = document.createElement("li");
				element.innerHTML = result;
				element.className = "list-group-item";
				list.appendChild(element);
			}
			
			document.getElementById("resultsContainer").style.display = "block";
		});
	}

	function upload() {
		let selectedURLs = [];
		for (let i = 0; i < selectedImages.length; i++) {
			selectedURLs.push(getOriginalURL(selectedImages[i].src));
		}
		console.log(selectedURLs);

		uploadOnIPFS(selectedURLs);
	}

	function updateCounter() {
		let allImages = getAllImages();

		let imageCountInfo = document.getElementById("imageCountInfo");
		imageCountInfo.textContent = `${selectedImages.length} ${selectedImages.length == 1 ? "image" : "images"} selected.`;
		let selectAllButton = document.getElementById("selectAllButton");
		selectAllButton.textContent = selectedImages.length == allImages.length ? "Deselect all" : "Select all";

		let uploadButton = document.getElementById("uploadButton");
		uploadButton.disabled = selectedImages.length == 0;

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

	/*function getImages(requestedURLs) {
		let imageContent = "";
		for (let i = 0; i < requestedURLs.length; i++) {
			imageContent += `<img src=\"${requestedURLs[i]}\" onclick=\"imageClicked\" style=\"opacity:50%\">\n`;
		}

		console.log(imageContent);
		return imageContent;
	}*/

	

	function loadImages() {
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
		})
		.then(function(res) {
			return res.json();
		})
		.then(function(resJson) {
			console.log("4");
			let data = resJson.data;
			let thumbnailURLs = [];
			for (let i = 0; i < data.length; i++) {
				imageURLs.push(data[i].link);
				thumbnailURLs.push(getThumbnailURL(data[i].link));
			}

			//document.getElementById("gallery").innerHTML = getImages(thumbnailURLs);
			//let gallery = buildGallery(["https://i.imgur.com/B0gv74Cm.jpg", "https://i.imgur.com/B0gv74Cm.jpg", "https://i.imgur.com/B0gv74Cm.jpg", "https://i.imgur.com/B0gv74Cm.jpg"], 400, 200, 10);
			let gallery = buildGallery(thumbnailURLs, 400, 200, 5);
			document.getElementById("galleryContainer").appendChild(gallery);
			console.log(resJson);
			return resJson;
		});
	}

  function buildGallery(imgURLs, galleryWidth, maxColumnWidth, gap) {
	let columnCount = parseInt(galleryWidth / maxColumnWidth) || 1;
	let columns = [];
	for (let i=0; i < columnCount; i++) {
		columns.push([]);
	}
	// Fill the columns with image URLs
	for (let i = 0; i<imgURLs.length; i++) {
		const idx = i % columnCount;
		columns[idx] = [...columns[idx] || [], imgURLs[i]];
	}

	let gallery = document.createElement("div");
	//gallery.clientWidth = 10;
	gallery.style = `grid-template-columns: repeat(${columnCount}, 1fr); --gap: ${gap}px`;
	gallery.id = "gallery";

	for (let i = 0; i < columnCount; i++) {
		let column = document.createElement("column");
		column.class = "column";

		for (let j = 0; j < columns[i].length; j++) {
			let url = columns[i][j];
			let image = document.createElement("img");
			image.src = url;
			image.alt = "";
			image.onclick = imageClicked;
			image.style.opacity = "50%";
			image.style.backgroundColor = "white";
			image.className = "imgurImage";
			column.appendChild(image);
		}
		gallery.appendChild(column);
	}
	console.log(gallery);
	return gallery;
  }

console.log(parsed);
  if (parsed.access_token != undefined) {
	  console.log("Loading images")
	  loadImages();
  }

</script>

<main>
	<h1>Imgur to IPFS</h1>
	<!--
	<button type="button" class="btn btn-primary btn-rounded">Light</button>
	-->
	{#if parsed.access_token != undefined}
		<p id="imageCountInfo">0 images selected.</p>
		<button id="selectAllButton" type="button" class="btn btn-secondary btn-rounded" on:click={toggleAll}>Select all</button>
		<button id="uploadButton" type="button" class="btn btn-primary btn-rounded" on:click={upload} disabled>Upload</button>
		<!--
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
		</div>
		-->
		<div id="resultsContainer" style="display:none">
			<p>Done! Here are your CIDs:</p>
			<ul id="cidList" class="list-group list-group-flush">

			</ul>
			<p>Your browser is now running an IPFS node, don't close it!</p>
		</div>
		<div id="galleryContainer">

		</div>
		<!--{@html buildGallery(["https://i.imgur.com/B0gv74Cm.jpg"], 400, 200, 5)}-->
		<!--
			<Gallery gap="10" onclickEvent={imageClicked} imageClass="imgurImage">
			<img src="https://i.imgur.com/B0gv74Cm.jpg" onclick="imageClicked" style="opacity:50%">
		</Gallery>
		-->
	{:else}
		<a href="{imgurUrl}" class="btn btn-primary" role="button" rel="noreferrer">Login with Imgur</a>
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