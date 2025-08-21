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

	let size = 500;
	let regenerate = false;
	let canvasElement = $state();
	let ctx;

	onMount(() => {
		ctx = canvasElement.getContext('2d');
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

		drawRect(0, 0, 100, 100, 'red');
		drawRect(200, 200, 100, 100, 'blue');
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
		const oldScale = viewportTransform.scale;
		const oldX = viewportTransform.x;
		const oldY = viewportTransform.y;

		const localX = e.clientX;
		const localY = e.clientY;

		const previousScale = viewportTransform.scale;

		const newScale = (viewportTransform.scale += e.deltaY * -0.01);
		
		const newX = localX - (localX - oldX) * (newScale / previousScale);
		const newY = localY - (localY - oldY) * (newScale / previousScale);

		viewportTransform.x = newX;
		viewportTransform.y = newY;
		viewportTransform.scale = newScale;
		console.log(viewportTransform.scale)
	}

	const onMouseWheel = (e) => {
		updateZooming(e);

		render();

		console.log(e);
	}

	const onMouseMove = (e) => {
		updatePanning(e);
		render();
		console.log(e);
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

	let value = $state();
	let input;

	async function sendData(y) {
		setDoc(doc(db, 'data', `row${y}`), {
			row: y,
			rowData: pixels[y],
			timestamp: Date.now()
		});
	}
</script>

<h1>Hello world!</h1>
<form onsubmit={sendData}><input type="text" bind:value bind:this={input} /></form>

<canvas
	bind:this={canvasElement}
	class="h-auto border"
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
