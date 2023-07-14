<script lang="ts">
	import { Geolocation } from '@capacitor/geolocation';
	import { Filesystem, Directory, Encoding } from '@capacitor/filesystem';
	import { FilePicker, type PickFilesResult } from '@capawesome/capacitor-file-picker';
	import ePub from 'epubjs';
	import { Capacitor } from '@capacitor/core';

	let book = ePub();
	let rendition: any;
	let path: any;
	let platform: string = Capacitor.getPlatform();

	let loc: { coords: { latitude: any; longitude: any } } | null = null;
	async function getCurrentPosition() {
		const res = await Geolocation.getCurrentPosition();
		loc = res;
	}
	function openBook(e: any) {
		const bookData = e.target.result;
		// console.log('bookData', bookData);
		book.open(bookData, 'binary');

		// console.log(book);
		rendition = book.renderTo('viewer', {
			width: '100%',
			height: 1200
		});
		rendition.display();
	}
	// let bookFile = Object();
	const readFile = async () => {
		const contents = await FilePicker.pickFiles({
			types: ['application/epub+zip'],
			readData: true,
			multiple: false
		}).catch((e) => {
			console.log('contents', e);
			// means that we didnt not select any files
			return null;
		});
		console.log(contents?.files[0]);
		if (contents !== null)
			switch (platform) {
				case 'web': {
					console.log('web');
					const file = contents.files[0];
					if (window.FileReader) {
						let reader = new FileReader();
						reader.onload = openBook;
						reader.readAsArrayBuffer(file?.blob!);
					}
					break;
				}
				case 'android': {
					console.log('android');
					const file = contents.files[0];
					// path = file.path;
					const b64 = createBase64Url(file.data);
					book.open(b64!);
					// book = ePub()
					// book = ePub(b64);
					// book.open(b64);
					// book.open(path);
					// const content = contents.files[0].data;
					// book = ePub(content!, { encoding: 'base64' });
					// const b64 = content?.replace('data:application/epub+zip;base64,', '');
					// book.open(b64);
					// alert(JSON.stringify(book, null, 2).toString());
					rendition = book.renderTo('viewer', {
						width: '100%',
						height: 1200
					});
					rendition.display();
					break;
				}
			}
	};
	function createBase64Url(content: any) {
		let data;
		let dataUri;

		if (typeof content !== 'string') {
			return;
		}
		data = btoa(encodeURIComponent(content));
		dataUri = 'data:' + 'application/epub+zip' + ';base64,' + data;
		return dataUri;
	}
	// if (platform === 'web') {
	// 	if (window.FileReader) {
	// 		let reader = new FileReader();
	// 		reader.onload = openBook;
	// 		reader.readAsArrayBuffer(bookFile.blob);
	// 	}
	// } else if (platform === 'android') {
	// 	path = bookFile.path;
	// 	// path = contents.files[0];
	// 	book = ePub(path);
	// 	alert(JSON.stringify(book, null, 2).toString());
	// 	rendition = book.renderTo('viewer', {
	// 		width: '100%',
	// 		height: '100%'
	// 	});
	// 	rendition.display();
	// }

	// console.log(contents.files[0]);
	// book.open(contents.files[0], 'binary');

	// const bin = await Filesystem.readFile({ path: path });
	// path = bin;
	// const file = contents.files[0];

	// const read = await readFile();
	// if (window.FileReader) {
	// 	let reader = new FileReader();
	// 	reader.onload = openBook;
	// 	reader.readAsArrayBuffer(file.blob!);
	// }
</script>

<div>
	<!-- <pre class="text-sky-400">{JSON.stringify(path, null, 2)}</pre> -->
	<!-- <h1>{platform}</h1> -->
	{#if platform === 'android' && path !== undefined}
		<p>{path}</p>
	{/if}
	<button class="btn btn-primary" on:click={readFile}> Read File </button>
	<div id="viewer" class="container" />
</div>
