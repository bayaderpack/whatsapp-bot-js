  <script>
import { qr } from '@svelte-put/qr/svg';
import * as XLSX from 'xlsx';
let phone = ''
	let message = ''
	let isReady = false
	let qrr = ''
	let image
	let showImage = false
	let input
	let imageMsg
	

		window.electron.receive('qrcode', (data) => {
		
			if(typeof data == "string" && data)
			qrr = data
		});

		window.electron.receive('ready', (data) => {
		
			isReady = data
	});
	const send = () => {
	  if (isReady ) {
		const phoneNumber = Array.from(phone)[0] == '+' ? phone.substring(1) : phone

		if(!showImage) {
		window.electron.send('sendMessage', {phone: `${phoneNumber}@s.whatsapp.net`, message: message});
		} else {
			window.electron.send('sendMessageImage', {phone: `${phoneNumber}@s.whatsapp.net`, message: message, image: imageMsg});
		}

	  }
	}

		// window.electron.send('ready', "asada");

		function onChange() {
    const file = input.files[0];

    if (file) {
			showImage = true;

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
console.log("fired")

  const file = e.dataTransfer.files[0];
  const data = await file.arrayBuffer();
  /* data is an ArrayBuffer */
  const workbook = XLSX.read(data);
  console.log(workbook)

  /* DO SOMETHING WITH workbook HERE */
}
  </script>
  
  <div class="wrapper">
	{#if qrr == '' && !isReady}
	  <div>Please wait program is loading</div>
	{:else if qrr != '' && !isReady}
	<!-- {qrr} -->
	<svg width="256px"
	use:qr={{
	  data: qrr,
	}}
  />
	{:else if qrr != '' && isReady}
	  <h1>Bayaderpack WhatsApp Bot Sender</h1>

	  {#if showImage}
	  <img bind:this={image} src="" alt="Preview" width="256px" />
	  {/if}
	  
	  <div id="drop"     on:drop={handleDrop } 
		on:dragover={(ev) => { ev.preventDefault() }}>Drop a spreadsheet file with numbers</div>
	  
	  <input type="text" class="input" placeholder="Phone number" bind:value={phone} />
	  <input type="text" class="input" placeholder="Message" bind:value={message} />
	  <input
	bind:this={input}
	on:change={onChange}
  type="file"
/>
	  <button on:click={send}>Send</button>
	{/if}
  </div> 

  <style>
	.wrapper {
		min-height: 100vh;
		width: 100vw;
		background-color: hsl(150, 90%, 67%);
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}
	.wrapper div{
		color: black;
		font-size: 3rem;
	}
	#drop {
		font-size:1.2rem;
		color: #eee;
		padding: 4rem;
		border: 2px solid #eee;
		
	}
	#drop:hover {
		cursor:grab;
	}
  </style>