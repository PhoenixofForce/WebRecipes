<script>
    //https://svelte.dev/repl/72d3ca877047468db8bb3a68f25a8b92?version=3.14.1
    import { onMount } from 'svelte';

    export let codeValue;
    export let squareSize; 

    export let useCompression;
    let whiteBackGround = false;

    let correctLevel = QRCode.CorrectLevel.L;
    let charactersPerLevel = [2950, 2328, 1660, 1270]
    //[4296, 3391, 2429, 1852]; alphanumeric
    //[2953, 2331, 1663, 1273]; binary/bits in theory
    //[2950, 2328, 1660, 1270]; binary/bits in praxis

    let qrcode;
    let loaded = false;

    let error = "";

    let ud = "" + Math.random();

    onMount(() => {
        loaded = true;
        qrcode = new QRCode(ud, {
            text: codeValue,
            width: squareSize,
            height: squareSize,
            colorDark : "#000",
            colorLight : whiteBackGround? "#FFF": getComputedStyle(document.body).getPropertyValue("--color-primary-light"),
            correctLevel : correctLevel
        });
    });

    $: updateCode(codeValue)
    $: setCorrectlevel(correctLevel)
    $: setColor(whiteBackGround)
    function setCorrectlevel(level) {
        if(loaded) {
            qrcode._htOption.correctLevel = valueToCorrectLevel(level); 
            console.log(level + "=>" + qrcode._htOption.correctLevel)
            qrcode.clear();
            updateCode(codeValue)
        }
    }

    function setColor(useWhite) {
        if(loaded) {
            qrcode._htOption.colorLight = useWhite? "#FFF": getComputedStyle(document.body).getPropertyValue("--color-primary-light");
            qrcode.clear();
            updateCode(codeValue)
        }
    }

    function updateCode(text)  {
        if(loaded) {
            qrcode.clear();
            try {
                qrcode.makeCode(text);
                error = " ";
            } catch(exception) {
                qrcode.makeCode("");
                error = "Input for the qr code is too long, please reduce your text."
            }
        }
    }

    function valueToCorrectLevel(level) {
        return (level == 0? 1: (level == 1? 0: (level == 2? 3: 2)));
    }
    </script>

<div>
    <div class="is-center is-horizontal-align"><div id={ud}></div></div>
    <div class="error">
        { error }
    </div>

    <!--
    <div style="color: {codeValue.length > 0.7 * charactersPerLevel[correctLevel]? "red": "gray"}">
        {
            (codeValue.at(0) == 'c'? "compressed": "uncompressed") + " at " + codeValue.length + " characters"
        }
    </div>
    -->

    <div class:error={codeValue.length > charactersPerLevel[correctLevel]}>
        <input bind:value={ correctLevel } type="range" min="0" max="3" list="values">
        <datalist id="values">
            <option value="0" label="Low"></option>
            <option value="1" label="Medium"></option>
            <option value="2" label="Quality"></option>
            <option value="3" label="High"></option>
        </datalist>
        <span>
            { "QRCodes with an CorrectnessLevel of " + correctLevel + " can hold up to " + charactersPerLevel[correctLevel] + " character (" 
            + codeValue.length + " characters, " + (100.0 * codeValue.length) / charactersPerLevel[correctLevel] + "% used)"  }
        </span>
    </div>

    <div>
        <label for="compression">
            <input id="compression" bind:checked={ useCompression } type=checkbox />
            Use Compressing <span class="error">(Warning: unsupported by the App)</span>
        </label>
    </div>

    <div>
        <label for="background">
            <input id="background" bind:checked={ whiteBackGround } type=checkbox />
            White Background for the QR Code(might help with scanning)
        </label>
    </div>
</div>

<style>
    #qrcode {
        max-width: 500px;
        width:100%;
        aspect-ratio: 1;
    }

    .error {
        color: var(--color-error);
        font-weight: bold;
    }

    div div {
        margin-bottom: 8px;
    }
</style>

