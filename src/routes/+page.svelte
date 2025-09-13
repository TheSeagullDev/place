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

	const colors = [
		'6d001a',
		'be0039',
		'ff4500',
		'ffa800',
		'ffd635',
		'fff8b8',
		'00a368',
		'00cc78',
		'7eed56',
		'00756f',
		'009eaa',
		'00ccc0',
		'2450a4',
		'3690ea',
		'51e9f4',
		'493ac1',
		'6a5cff',
		'94b3ff',
		'811e9f',
		'b44ac0',
		'e4abff',
		'de107f',
		'ff3881',
		'ff99aa',
		'6d482f',
		'9c6926',
		'ffb470',
		'000000',
		'515252',
		'898d90',
		'd4d7d9',
		'ffffff'
	];

	let size = 1000;
	let regenerate = false;
	let canvasElement = $state();
	let hoverOverlay = $state();
	let ctx;
	let hoverCtx;
	let imageData;

	const imageArr = new Uint8ClampedArray(size * size * 4);

	let hoveredCoords = [];
	let prevHover = [];

	for (let row = 0; row < size; row++) {
		for (let col = 0; col < size; col++) {
			const color = colors[Math.floor(Math.random() * colors.length)];
			const r = parseInt(color.slice(0, 2), 16);
			const g = parseInt(color.slice(2, 4), 16);
			const b = parseInt(color.slice(4, 6), 16);
			const index = row * (size * 4) + col * 4;
			imageArr[index] = r;
			imageArr[index + 1] = g;
			imageArr[index + 2] = b;

			imageArr[row * (size * 4) + col * 4 + 3] = 255;
		}
	}

	onMount(() => {
		ctx = canvasElement.getContext('2d');
		hoverCtx = hoverOverlay.getContext('2d');
		ctx.imageSmoothingEnabled = false;
		render();
	});

	const viewportTransform = {
		x: 0,
		y: 0,
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
				drawPixel(row, col, imageArr[row][col]);
			}
		}
	};

	const render = () => {
		ctx.setTransform(1, 0, 0, 1, 0, 0);
		ctx.clearRect(0, 0, canvasElement.width, canvasElement.height);
		ctx.fillStyle = '#808080';
		ctx.fillRect(0, 0, size, size);

		ctx.setTransform(
			viewportTransform.scale,
			0,
			0,
			viewportTransform.scale,
			viewportTransform.x,
			viewportTransform.y
		);

		const imageData = new ImageData(imageArr, size, size);
		let imageBitmap;

		createImageBitmap(imageData).then((bitmap) => {
			imageBitmap = bitmap;
			ctx.drawImage(imageBitmap, 0, 0);
		});
	};

	const renderHover = () => {
		const thickness = 0.2;
		hoverCtx.setTransform(1, 0, 0, 1, 0, 0);
		hoverCtx.clearRect(0, 0, hoverOverlay.width, hoverOverlay.height);
		hoverCtx.setTransform(
			viewportTransform.scale,
			0,
			0,
			viewportTransform.scale,
			viewportTransform.x,
			viewportTransform.y
		);
		const coords = getIndicesForCoord(hoveredCoords[0], hoveredCoords[1], size);
		const r = imageArr[coords[0]].toString(16).padStart(2, '0');
		const g = imageArr[coords[1]].toString(16).padStart(2, '0');
		const b = imageArr[coords[2]].toString(16).padStart(2, '0');

		const luma = 0.2126 * r + 0.7152 * g + 0.0722 * b;
		const color = `#${r}${g}${b}`;
		if (luma < 40) {
			hoverCtx.fillStyle = '#FFFFFF';
		} else {
			hoverCtx.fillStyle = '#000000';
		}

		hoverCtx.fillRect(hoveredCoords[0], hoveredCoords[1], 1, 1);
		hoverCtx.fillStyle = color;
		hoverCtx.fillRect(
			hoveredCoords[0] + thickness / 2,
			hoveredCoords[1] + thickness / 2,
			1 - thickness,
			1 - thickness
		);
	};

	const clearHover = () => {
		hoverCtx.setTransform(1, 0, 0, 1, 0, 0);
		hoverCtx.clearRect(0, 0, hoverOverlay.width, hoverOverlay.height);
		hoverCtx.setTransform(
			viewportTransform.scale,
			0,
			0,
			viewportTransform.scale,
			viewportTransform.x,
			viewportTransform.y
		);
	};

	let panning = false;
	let previousX = 0,
		previousY = 0;

	const updatePanning = (e) => {
		const localX = e.clientX;
		const localY = e.clientY;

		viewportTransform.x += localX - previousX;
		viewportTransform.y += localY - previousY;

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
		let screenScaleX = size / bounding.width;
		let screenScaleY = size / bounding.height;
		let x = (event.clientX - bounding.left) * screenScaleX;
		let y = (event.clientY - bounding.top) * screenScaleY;
		x = Math.floor((x - viewportTransform.x) / viewportTransform.scale);
		y = Math.floor((y - viewportTransform.y) / viewportTransform.scale);
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

<div class="fixed left-[12.5%] h-screen w-3/4">
	<canvas
		bind:this={canvasElement}
		class="absolute h-11/12"
		onmousedown={(e) => {
			previousX = e.clientX;
			previousY = e.clientY;

			panning = true;
		}}
		onmousemove={(e) => {
			if (panning) {
				onMouseMove(e);
				hoverCtx.setTransform(1, 0, 0, 1, 0, 0);
				hoverCtx.clearRect(0, 0, hoverOverlay.width, hoverOverlay.height);
			} else {
				prevHover = hoveredCoords;
				hoveredCoords = pick(e);
				if (
					hoveredCoords[0] >= 0 &&
					hoveredCoords[0] < 1000 &&
					hoveredCoords[1] >= 0 &&
					hoveredCoords[1] < 1000
				) {
					renderHover();
				} else {
					clearHover();
				}
			}
		}}
		onmouseup={() => (panning = false)}
		onwheel={(e) => {
			hoverCtx.setTransform(1, 0, 0, 1, 0, 0);
			hoverCtx.clearRect(0, 0, hoverOverlay.width, hoverOverlay.height);
			onMouseWheel(e);
		}}
		width={size}
		height={size}
	></canvas>
	<canvas
		bind:this={hoverOverlay}
		class="[pointer-events:none] absolute h-11/12"
		width={size}
		height={size}
	>
	</canvas>
</div>
