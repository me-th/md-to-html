<script>
	import { copy } from 'svelte-copy';
    import { text } from 'svelte/internal';
	let copyButton = "copy it";
	let markdown = '';
	let htmlOutput = '';
	let lines = "";
	function addHeading(text, value=0){
		if(text[0] == "#"){
			if(value < 6){
				let param = value + 1;
				return addHeading(text.slice(1), param);
			}else{
				return "######" + text;
			}
		}else if(text[0] == " "){
			let headingStart = "<h"+value+">";
			let headingEnd = "</h"+value+">";
			let headingContent = text.slice(1);
			let parsedText = headingStart + headingContent + headingEnd;
			return parsedText;
		}else{
			return text;
		}
	}
	function checkForHyperlinks(text){
		if(text.includes("[") && text.includes("]")){
			if(text.includes("(") && text.includes(")")){
				return true
			}
		}
	}
	function isItImageOrHyperlink(text){
		if(text.includes("!")){
			if(text.indexOf("!") > text.indexOf("[")){
				return "hyperlinkimage";
			}else{
				return "image";
			}
		}else{
			return "hyperlink";
		}
	}
	function convertImage(text){
		let hyperLinkStart = text.indexOf("(");
		let hyperLinkEnd = text.indexOf(")");
		let altTextStart = text.indexOf("[");
		let altTextEnd = text.indexOf("]");
		let extractedLink = text.slice(hyperLinkStart + 1, hyperLinkEnd);
		let extractedAltText = text.slice(altTextStart + 1, altTextEnd);
		let htmlTag = '<img src="' + extractedLink + '" alt="' + extractedAltText + '">'
		return htmlTag
	}
	function convertLink(text){
		let hyperLinkStart = text.indexOf("(");
		let hyperLinkEnd = text.indexOf(")");
		let hyperTextStart = text.indexOf("[");
		let hyperTextEnd = text.indexOf("]");
		let extractedLink = text.slice(hyperLinkStart + 1, hyperLinkEnd);
		let extractedText = text.slice(hyperTextStart + 1, hyperTextEnd);
		let htmlTag = '<a href="' + extractedLink + '">' + extractedText + '</a>'
		return htmlTag
	}

	function convertHyperlinkImage(text){
		let temp = text.indexOf("[") + 1;
		let temp2 = text.indexOf(")") + 1;
		let extractedLink = text.slice((temp2 + 2), text.length-1);
		let extractedImage = text.slice(temp, temp2);
		let imageCode = convertImage(extractedImage);
		let htmlTag = '<a href="' + extractedLink + '">' + imageCode + '</a>';
		return htmlTag;
	}

	function checkInput(text){
		if(text.length >= 1){
			if(text[0] == "#"){
				let output = addHeading(text);
				return output;
			}else if(checkForHyperlinks(text)){
				let type = isItImageOrHyperlink(text);
				if(type == "image"){
					let output = "";
					output = convertImage(text);
					return output;
				}else if(type == "hyperlink"){
					let output = "";
					output = convertLink(text);
					return output;
				}else{
					let output = "";
					output = convertHyperlinkImage(text);
					return output
				}
			}else if((text[0] == "<") && (text[text.length-1] == ">")){
				return text
			}
			else{
				let output = "<p>" + text + "</p>";
				return output;
			}
		}
		return "";
	}
	function buttonState(){
		copyButton = "copied!";
		setTimeout(function changeState(){copyButton = "copy it"}, 2000)
	}
	$: lines = markdown.split(/\r?\n/);
	$: htmlOutput = lines.map(line => checkInput(line)).join('\n');
</script>

<h3>type markdown here</h3><br>
<textarea name="markdownInput" id="markdownInput" cols="30" rows="10" bind:value={markdown} placeholder="type something!"></textarea>
<br>
<h3>html output</h3><button use:copy={htmlOutput} on:click={buttonState}>{copyButton}</button><br>
<textarea name="htmlOutput" id="output" cols="30" rows="10" value={htmlOutput} disabled></textarea>
<div class="half">
	<h3>live preview</h3>
	<div class="preview">
		{@html htmlOutput}
	</div>
</div>

<style>
	.half {
		position: fixed;
		width: 1400px;
		top: 0;
		height: 100%;
		right: 10px;
	}
	.preview {
		padding: 5px;
		border: 3px solid black;
		height: 90%;
	}
</style>