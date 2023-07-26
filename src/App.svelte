<script>
	import QRCode from "./QRJS.svelte"

	let allowCompression = false;

	let ingredientInput;
	let recipe = {
		name: "",
		portions: 4,
		time: "",
		description: "",
		directions: "",
		ingredients: [],
		tags: ""
	}

	//https://gist.github.com/hyamamoto/fd435505d29ebfa3d9716fd2be8d42f0
	function hashCode(s) {
		return [...s].reduce(
			(hash, c) => (Math.imul(31, hash) + c.charCodeAt(0)) | 0,
			0
		);
	}

	let qrCodeInput = toAndroidFormat(recipe);
	let qrCodeHashInput = toAndroidHash(recipe);
	$: {
		recipe;
		allowCompression;
		debounce();
	}

	function toAndroidHash(recipeData) {
		
		let out = 17;

		out = out*31 + hashCode(recipeData.name);
		out = out*31 + hashCode(recipeData.portions + "");
		out = out*31 + hashCode(recipeData.time);
		for(let ingredient of recipeData.ingredients) {
			let internHash = 17;

			internHash = internHash*31 + hashCode(ingredient.name);
			internHash = internHash*31 + hashCode(ingredient.unit);
			internHash = internHash*31 + hashCode(ingredient.amount + "");


			//out = out*31 + (internHash|0);	//this somehow ruins it
		}
		out = out*31 + hashCode(recipeData.description);
		out = out*31 + hashCode(recipeData.directions);

		return "" + (out|0);
	}

	function toAndroidFormat(recipeData) {
		let out = {};

		out["name"] = recipeData.name;
		out["portions"] = recipeData.portions;
		if(recipeData.time.length > 0) out["time"] = recipeData.time;
		if(recipeData.description.length > 0) out["description"] = recipeData.description;
		if(recipeData.directions.length > 0) out["directions"] = recipeData.directions;
		out["ingredientCount"] = recipeData.ingredients.length;
		if(recipeData.tags.length > 0) out["tags"] = recipeData.tags;

		for(let i = 0; i < recipeData.ingredients.length; i++) {
			out["ingredient_" + i] = recipeData.ingredients.at(i);
		}
		return compress(JSON.stringify(out));
	}

	function compress(string) {
		string = string.replace(/\s+/i, " ")

		var compressed = LZString.compressToEncodedURIComponent(string);
		console.log("compression would reduce " + string.length + " to " + compressed.length);

		let out = "";
		if(allowCompression) out += (compressed.length < string.length && allowCompression)? ("c"): ("u")
		out += (compressed.length < string.length && allowCompression)? (compressed): (string);
		return out;
	}

	function textToIngredient(textIn) {
		if(! textIn.match(/\d+(.\d+)? .+ .+/i)) return;

		let parts = textIn.split(" ");
		let amount = parseFloat(parts[0]);
		let unit = parts[1];

		let name = "";
		for(let i = 2; i < parts.length; i++) name += parts[i];

		recipe.ingredients.push({amount: amount, unit: unit, name: name});
		recipe.ingredients = recipe.ingredients;
		ingredientInput = "";
	}

	let timer;
	const debounce = e => {
		clearTimeout(timer);
		timer = setTimeout(() => {
			qrCodeInput = toAndroidFormat(recipe);
			qrCodeHashInput = toAndroidHash(recipe)
		}, 1000);
	}
</script>

<div class="main">
	<div class="row">
		<div class="col">
			<div>
				<label for="name">Recipe Name</label>
				<input id="name" type="text" bind:value={ recipe.name } placeholder="Recipe Name"/>

				<label for="portions">Portions</label>
				<input id="portions" type="number" bind:value={ recipe.portions } placeholder="Portions"/>

				<label for="time">Cooking Time</label>
				<input id="time" type="text" bind:value={ recipe.time } placeholder="Time" />

				<label for="description">Description</label>
				<input id="description" type="text" bind:value={ recipe.description } placeholder="Description"/>

				<label for="directions">Cooking Directions</label>
				<textarea id="directions" rows="7" bind:value={ recipe.directions } placeholder="Cooking Directions"/>

				<label for="ingredients">Ingredients</label>
				<ul>
					{#each recipe.ingredients as ingredient, i}
					<li>
						{ingredient.amount + ingredient.unit + " " + ingredient.name}
						<button class="button icon-only pull-right"
							on:click={ e => { recipe.ingredients.splice(i, 1); recipe.ingredients = recipe.ingredients; } }>
							x
						 </button>
					</li>
				{/each}
				</ul>
				<input id="ingredients" bind:value={ ingredientInput } type="text" placeholder="Amount Unit Name (ex: '150 g Flour')" 
					on:keydown={e => { if(e.key == 'Enter') textToIngredient(ingredientInput)}} />

				<label for="tags">Tags</label>
				<input id="tags" type="text" bind:value={ recipe.tags } placeholder="tag1,tag2,tag3"/>
			</div>
		</div>

		<div class="col">
			<QRCode bind:codeValue={ qrCodeInput } bind:useCompression={ allowCompression } squareSize=1000 />
		</div>

		<div class="col">
			<QRCode bind:codeValue={ qrCodeHashInput } bind:useCompression={ allowCompression } squareSize=1000 />
		</div>

	</div>
</div>



<style>
	.main {
		width: 90%;
		margin-left: auto;
	  	margin-right: auto;
	}

	.row {
		margin-top: 50px;
	}

	input, textarea{
		margin-bottom: 8px;
	}

	.button {
		transform: translate(0, -25%);
		padding-top: 8px;
	}

	li {
		margin-bottom: 16px;
	}

	label {
		font-weight: 400;
	}
</style>
