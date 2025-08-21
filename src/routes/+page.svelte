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

	let size = 100;
	let regenerate = false;
	let canvasElement;
	let ctx;

	onMount(() => {
		ctx = canvasElement.getContext('2d');
	})

	function getIndicesForCoord(x, y, width) {
		const r = y * (width * 4) + x * 4;
		return [r, r + 1, r + 2, r + 3];
	}


	/* for (let i = 0; i < size; i++) {
		pixels.push(new Array(size));
	}

	for (let i = 0; i < pixels.length; i++) {
		for (let j = 0; j < pixels[i].length; j++) {
			pixels[i][j] = "FFFFFF";
		}
		if(regenerate) {
			sendData(i);
		}
	} */

	console.log(pixels);

	let value = $state();
	let input;

	async function sendData(y) {
		setDoc(doc(db, 'data', `row${y}`), {
			row: y,
			rowData: pixels[y],
			timestamp: Date.now()
		});
	}

/* 	const q = query(collection(db, 'data'));
	onSnapshot(q, (snaps) => {
		console.log(snaps);
		snaps.forEach((doc) => {
			const rowData = doc.data().rowData;
			pixels[doc.data().row] = rowData;
		});
	}); */
</script>

<h1>Hello world!</h1>
<form onsubmit={sendData}><input type="text" bind:value bind:this={input} /></form>

<canvas bind:this={canvasElement}></canvas>
