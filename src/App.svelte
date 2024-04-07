<script >
	import { qr } from '@svelte-put/qr/svg';
	// import * as XLSX from 'xlsx';
	import { read, utils } from 'xlsx';
		let message = ''
		let isReady = false
		let qrr = ''
		let image
		let showImage = false
		let input
		let imageMsg
		let allNumbers = undefined
		let dropMsg = "Drop a spreadsheet file with numbers"
		let imageButton = "Choose an image"
		let errMsg = ''
		let errShow = false
		let dialog
		
	
			window.electron.receive('qrcode', (data) => {
			
				if(typeof data == "string" && data)
				qrr = data
			});
	
			window.electron.receive('ready', (data) => {
			
				isReady = data
		});
		const send = () => {
		  if (isReady && allNumbers !== undefined) {
			let promise = new Promise(function(resolve, reject) {
    dialog.showModal();
    dialog.onclose = resolve;
  });
  promise.then(function(response) {
    if(response.target.returnValue == "ok") {
		allNumbers.forEach(num => {
		const phoneNumber = Array.from(num.Phone)[0] == '+' ? num.Phone.substring(1) : num.Phone
		if(!showImage) {
			const formatedMsg = message.replaceAll("{name}", num.Name)
			window.electron.send('sendMessage', {phone: `${phoneNumber}@s.whatsapp.net`, message: formatedMsg});
			} else {
				window.electron.send('sendMessageImage', {phone: `${phoneNumber}@s.whatsapp.net`, message: message, image: imageMsg});
			}
			wait(2000)
	  })
	}
  });

	
	
		  }
		  else {
			errShow = true;
			errMsg = "You need to add numbers!"
		  }
		}
	
			// window.electron.send('ready', "asada");
	
			function onChange() {
		const file = input.files[0];
	
		if (file) {
				showImage = true;
	imageButton = file.name
		  const reader = new FileReader();
		  reader.addEventListener("load", function () {
			imageMsg = reader.result.split(',')[1];
			
			image.setAttribute("src", reader.result);
		  });
		  reader.readAsDataURL(file);
				
				return;
		} 
			showImage = false; 
	  }
		
	
	  async function handleDrop(e) {
	
	  const file = e.dataTransfer.files[0];
	 dropMsg = file.name
	  const data = await file.arrayBuffer();
	
	  const workbook = read(data);
	allNumbers = utils.sheet_to_json(workbook.Sheets[workbook.SheetNames[0]])
	}

	  
	const closeError = () => {
		errMsg = ''
		errShow = false
	}

	function wait(ms){
   var start = new Date().getTime();
   var end = start;
   while(end < start + ms) {
     end = new Date().getTime();
  }
}
 </script>
 <div class="wrapper">
	{#if  errMsg != '' && errShow}
		<div class="error-wrapper">
			<div class="error">
				<button class="error-button" on:click={closeError}>
					X
				</button>
				<p>{errMsg}</p>
			</div>
		</div>
	{/if}
	{#if qrr == '' && !isReady}
	   <div>Please wait program is loading</div>
	   {:else if qrr != '' && !isReady}
	   
	   <svg width="256px"
	   use:qr={{
	   data: qrr,
	   }}
	   />
	   {:else if qrr != '' && isReady}
	<h1>Bayaderpack WhatsApp Bot Sender</h1>
	<div class="inner-wrapper">
	   {#if showImage}
	   <img bind:this={image} src="" alt="Preview" width="256px" />
	   {/if}
	   <div id="drop" class="input"     on:drop={handleDrop } 
		  on:dragover={(ev) => { ev.preventDefault() }}>{dropMsg}</div>
	   <textarea type="text" class="input" rows="3" placeholder="Message" bind:value={message} />
	   <input type="file" id="file" bind:this={input} on:change={onChange} class="inputfile" />
	   <label for="file">{imageButton}</label>
	   <button on:click={send}>Send</button>
	</div>
	{#if allNumbers !== undefined}
	<dialog bind:this={dialog}>
		<form method="dialog">
		  <p>Send message to {allNumbers.length} numbers</p>
		  <div class="dialog-buttons">
			<button value="ok">Ok</button>
			<button value="cancel" id="error-button">No</button>
		  </div>

		</form>
	  </dialog>
	{/if}

	{/if}
 </div>
 <style>
	:root {
  --ratio: 3.74;
  --error-color: rgb(214, 40, 16);
  --error-color-hover: rgb(168, 30, 12);
  --light-color-bg: rgb(238, 241, 247);
  --dark-color-bg: rgb(55, 65, 81);
  --grey-color: rgb(107, 114, 128);
  --leading: 0.025em;
}
	* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
	}
	#error-button {
		background-color: var(--error-color);
		color: var(--light-color-bg);
	}
	#error-button:hover {
		background-color: var(--error-color-hover);
		color: var(--light-color-bg);
	}
	.error-wrapper {
		position: fixed;
		inset: 0 0 0 0;
		background-color: rgba(0,0,0,0.4);
		display: flex;
		justify-content: center;
		align-items: center;

	}
	dialog {
  border: none !important;
  border-radius: calc(5px * 3.74);
  box-shadow: 0 0 #0000, 0 0 #0000, 0 25px 50px -12px rgba(0, 0, 0, 0.25);
  padding: 1.6rem;
  max-width: 600px;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
	.error {
		border: none !important;
  border-radius: calc(5px * 3.74);
  box-shadow: 0 0 #0000, 0 0 #0000, 0 25px 50px -12px rgba(0, 0, 0, 0.25);
  padding: 1.6rem;
  max-width: 600px;
  position: relative;
  /* top: 50%;
  left: 50%;
  transform: translate(-50%, -50%); */
  background: var(--light-color-bg);
	}
	.error p {
		color: var(--error-color);
		font-size: 1.3rem;
		font-weight: 900;
	}
	.dialog-buttons {
		display: flex;
		justify-content: space-between;
	}
	button.error-button {
		position: absolute;
		width: 50px;
		height: 50px;
		top: -60px;
		border: 1px solid var(--error-color);
		right: 10px;
		border-radius: 50%;
	}

	 button {
  display: block;
  margin-top: 2rem;
  width: calc(22px * 3.74);
  height: 44px;
  border-radius: calc(3px * 3.74);
  border: none;
  letter-spacing: calc(3 *  0.025em);
  font-family: inherit;
  color: rgb(107, 114, 128);
  background-color: rgb(238, 241, 247);
  font-size: large;
  font-weight: 700;
}
button:hover {
	background-color: var(--grey-color);
	color: var(--light-color-bg);
}
	.wrapper {
	min-height: 100vh;
	width: 100vw;
	background-color: hsl(150, 60%, 45%);
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	gap: 2rem;
	}
	.wrapper h1{
	color: black;
	font-size: 2rem;
	}
	#drop {
	font-size:1.2rem;
	color: #000;
	border: 2px solid #eee;
	}
	#drop:hover {
	cursor:grab;
	}
	.input {
	appearance: none;
	padding: 15px 20px;
	border-radius: 2rem;
	width: 100%;
	margin: 0;
	}
	.inner-wrapper {
	width: 30rem;
	display: flex;
	flex-direction: column;
	gap: 0.7rem;
	align-items: center;
	}
	.inputfile {
	width: 0.1px;
	height: 0.1px;
	opacity: 0;
	overflow: hidden;
	position: absolute;
	z-index: -1;
	}
	.inputfile + label {
	font-size: 1.25em;
	font-weight: 700;
	color: white;
	background-color: black;
	display: inline-block;
	min-width: 12rem;
	max-width: fit-content;
	border-radius: 1rem;
	padding: 8px 24px;
	}
	.inputfile:focus + label,
	.inputfile + label:hover {
	background-color: var(--error-color);
	cursor: pointer; 
	}
	.inputfile:focus + label {
	outline: 1px dotted #000;
	outline: -webkit-focus-ring-color auto 5px;
	}
	.inputfile + label * {
	pointer-events: none;
	}

 </style>