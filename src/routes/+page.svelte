<script>
	import '../app.css';
	import {
		getFirestore,
		collection,
		doc,
		setDoc,
		addDoc,
		query,
		orderBy,
		onSnapshot
	} from 'firebase/firestore';
	import { initializeApp } from 'firebase/app';
	import { onMount } from 'svelte';

	const firebaseConfig = {
		apiKey: 'AIzaSyAnPWKF42Qz68nxsm1y-l58JPOrIgjMF94',
		authDomain: 'place-1f432.firebaseapp.com',
		projectId: 'place-1f432',
		storageBucket: 'place-1f432.firebasestorage.app',
		messagingSenderId: '560201387317',
		appId: '1:560201387317:web:0eb6bfe9839df3ce60907c'
	};

	const app = initializeApp(firebaseConfig);
	const db = getFirestore(app);

	let size = 1000;
	let regenerate = false;
	let canvasElement = $state();
	let ctx;
	let imageData;

	const randomArr = new Uint8ClampedArray(size * size * 4);

	let count = 0;

	for (let row = 0; row < size; row++) {
		for (let col = 0; col < size; col++) {
			for (let i = 0; i < 3; i++) {
				const index = row * (size * 4) + col * 4 + i;
				randomArr[index] = Math.floor(Math.random() * 255);
			}
			randomArr[row * (size * 4) + col * 4 + 3] = 255;
			count++;
		}
		if (count > 999900) {
			console.log('almost done!');
		}
	}
	console.log(count);

	onMount(() => {
		ctx = canvasElement.getContext('2d');
		ctx.imageSmoothingEnabled = false;
		render();
	});

	const viewportTransform = {
		x: 0,
		y: 0,
		scaledX: 0,
		scaledY: 0,
		scale: 1
	};

	const drawRect = (x, y, width, height, color) => {
		ctx.fillStyle = color;
		ctx.fillRect(x, y, width, height);
	};

	const drawPixel = (x, y, color) => {
		ctx.fillStyle = color;
		ctx.fillRect(x, y, 1, 1);
	};

	const paintScreen = () => {
		for (let row = 0; row < size; row++) {
			for (let col = 0; col < size; col++) {
				drawPixel(row, col, randomArr[row][col]);
			}
		}
	};

	const render = () => {
		ctx.setTransform(1, 0, 0, 1, 0, 0);
		ctx.clearRect(0, 0, canvasElement.width, canvasElement.height);


		ctx.setTransform(
			viewportTransform.scale,
			0,
			0,
			viewportTransform.scale,
			viewportTransform.x,
			viewportTransform.y
		);

		const imageData = new ImageData(randomArr, size, size);
		let imageBitmap;

		createImageBitmap(imageData).then((bitmap) => {
			imageBitmap = bitmap;
			ctx.drawImage(imageBitmap, 0, 0);
		});

		console.log(`scale: ${viewportTransform.scale} x: ${viewportTransform.x} y: ${viewportTransform.y} scaled x ${viewportTransform.x / (viewportTransform.scale - 1)}`);
	};

	let panning = false;
	let previousX = 0,
		previousY = 0;

	const updatePanning = (e) => {
		const localX = e.clientX;
		const localY = e.clientY;

		viewportTransform.x += localX - previousX;
		viewportTransform.scaledX = viewportTransform.x / (viewportTransform.scale - 1);
		if(viewportTransform.scaledX < -1000) {
			viewportTransform.x = -1000 * (viewportTransform.scale - 1);
		}
		else if(viewportTransform.x > 0) {
			viewportTransform.x = 0;
		}
		viewportTransform.y += localY - previousY;
		viewportTransform.scaledY = viewportTransform.y / (viewportTransform.scale - 1);
		if(viewportTransform.scaledY < -1000) {
			viewportTransform.y = -1000 * (viewportTransform.scale - 1);
		}
		else if(viewportTransform.y > 0) {
			viewportTransform.y = 0;
		}

		previousX = localX;
		previousY = localY;
	};

	const updateZooming = (e) => {
		const bounding = canvasElement.getBoundingClientRect();
		const oldScale = viewportTransform.scale;
		const oldX = viewportTransform.x;
		const oldY = viewportTransform.y;

		const localX = e.clientX - bounding.left;
		const localY = e.clientY - bounding.top;

		const previousScale = viewportTransform.scale;

		const newScale = (viewportTransform.scale += e.deltaY * -0.01);

		const newX = localX - (localX - oldX) * (newScale / previousScale);
		const newY = localY - (localY - oldY) * (newScale / previousScale);

		viewportTransform.x = newX;
		viewportTransform.y = newY;
		viewportTransform.scale = newScale;
		if (viewportTransform.scale < 1) {
			viewportTransform.x = oldX;
			viewportTransform.y = oldY;
			viewportTransform.scale = 1;
		}

		viewportTransform.scaledX = viewportTransform.x / (viewportTransform.scale - 1);
		if(viewportTransform.scaledX < -1000) {
			viewportTransform.x = -1000 * (viewportTransform.scale - 1);
		}
		else if(viewportTransform.x > 0) {
			viewportTransform.x = 0;
		}
		viewportTransform.y += localY - previousY;
		viewportTransform.scaledY = viewportTransform.y / (viewportTransform.scale - 1);
		if(viewportTransform.scaledY < -1000) {
			viewportTransform.y = -1000 * (viewportTransform.scale - 1);
		}
		else if(viewportTransform.y > 0) {
			viewportTransform.y = 0;
		}
		
	};

	const onMouseWheel = (e) => {
		updateZooming(e);

		render();
	};

	const onMouseMove = (e) => {
		updatePanning(e);
		render();
	};

	function getIndicesForCoord(x, y, width) {
		const r = y * (width * 4) + x * 4;
		return [r, r + 1, r + 2, r + 3];
	}

	const pick = (event) => {
		const bounding = canvasElement.getBoundingClientRect();
		let x = event.clientX - bounding.left;
		let y = event.clientY - bounding.top;
		x = Math.floor(x / scaleBy);
		y = Math.floor(y / scaleBy);
		return [x, y];
	};

	async function sendData(y) {
		setDoc(doc(db, 'data', `row${y}`), {
			row: y,
			rowData: pixels[y],
			timestamp: Date.now()
		});
	}
</script>

<button
	class="fixed"
	onclick={() => {
		viewportTransform.x = 0;
		viewportTransform.y = 0;
		viewportTransform.scale = 1;
		render();
	}}>Reset view</button
>

<div class="h-scren">
	<canvas
		bind:this={canvasElement}
		class="m-auto h-11/12 p-4"
		style="image-rendering: pixelated;"
		onmousedown={(e) => {
			previousX = e.clientX;
			previousY = e.clientY;
	
			panning = true;
		}}
		onmousemove={(e) => {
			if (panning) {
				onMouseMove(e);
			}
		}}
		onmouseup={() => (panning = false)}
		onwheel={(e) => onMouseWheel(e)}
		width={size}
		height={size}
	></canvas>
</div>
