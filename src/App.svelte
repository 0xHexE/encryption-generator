<script lang="ts">
	let certificateNo = '';
	let name = '';
	let workshopName = '';
	let venue = '';
	let startDate = '';
	let endDate = '';
	let privateKey = ``;

/*
Convert a string into an ArrayBuffer
from https://developers.google.com/web/updates/2012/06/How-to-convert-ArrayBuffer-to-and-from-String
*/
function str2ab(str) {
  const buf = new ArrayBuffer(str.length);
  const bufView = new Uint8Array(buf);
  for (let i = 0, strLen = str.length; i < strLen; i++) {
    bufView[i] = str.charCodeAt(i);
  }
  return buf;
}

/*
Import a PEM encoded RSA private key, to use for RSA-PSS signing.
Takes a string containing the PEM encoded key, and returns a Promise
that will resolve to a CryptoKey representing the private key.
*/
function importPrivateKey(pem) {
  // fetch the part of the PEM string between header and footer
  const pemHeader = "-----BEGIN PRIVATE KEY-----";
  const pemFooter = "-----END PRIVATE KEY-----";
  const pemContents = pem.substring(pemHeader.length, pem.length - pemFooter.length);
  // base64 decode the string to get the binary data
  const binaryDerString = window.atob(pemContents);
  // convert from a binary string to an ArrayBuffer
  const binaryDer = str2ab(binaryDerString);

  return window.crypto.subtle.importKey(
    "pkcs8",
    binaryDer,
    {
      name: "RSA-PSS",
      hash: "SHA-256",
    },
    true,
    ["sign"]
  );
}


	async function generateQR() {
		const enc = new TextEncoder(); // always utf-8

		const key = await importPrivateKey(privateKey.trim());

		const json = JSON.stringify({
			certificateNo,
			name,
			workshopName,
			venue,
			startDate,
			endDate,
		});

		const data = await window.crypto.subtle.sign(
			{
        name: "RSA-PSS",
        saltLength: 32,
      },
      key,
      enc.encode(json)
		);

		const finalData = btoa(String.fromCharCode(...new Uint8Array(data)));

		url = `${urlToVerify}?token=${ encodeURIComponent(finalData) }`

		const qr = new QRious({
			element: canvasElement,
			value: url,
			size: 512
		});
	}

	let canvasElement;
	let url;
	let urlToVerify = 'https://dypatilmedicalkop.org/verify-certifcate';
</script>

<form action="" on:submit|preventDefault={generateQR}>
	<div class="flex">
		<input bind:value={certificateNo} placeholder="Certificate No">
		<input bind:value={name} placeholder="Name of Participants">
		<input bind:value={workshopName} placeholder="Workshop name">
		<input bind:value={venue} placeholder="Venue">
		<input bind:value={startDate} placeholder="Start date" type="date">
		<input bind:value={endDate} placeholder="End date" type="date">
		<input bind:value={urlToVerify} type="url" placeholder="Website URL">
		<textarea bind:value={privateKey} placeholder="Private Key"></textarea>	
	</div>

	<button>
		Generate QR code
	</button>
</form>

<canvas bind:this={canvasElement}></canvas>

<br>

{#if url}
<a href={url}>Link for QR</a>
{/if}

<style>
	.flex {
		display: flex;
		flex-direction: column;
	}

	canvas {
		width: 512px;
		height: 512px
	}
</style>
