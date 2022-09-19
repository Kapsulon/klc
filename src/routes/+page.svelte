<script lang="ts">
    import { onMount } from 'svelte';
    import * as fs from '@tauri-apps/api/fs';
    import { resourceDir } from '@tauri-apps/api/path';
    import ColorPicker from "svelte-awesome-color-picker";

    const DEFAULT_COLORS:string[] = ["#46afc6", "#3498ad", "#2b7b8c"];

    let loaded:boolean = false;
    let hex:string = DEFAULT_COLORS[0];
    let selected_path:number = 0;

    onMount(async () => {
        loaded = true;
    });

    function componentToHex(c:number):string {
        var hex = c.toString(16);
        return hex.length == 1 ? "0" + hex : hex;
    }

    function rgbToHex(r:number, g:number, b:number):string {
        return "#" + componentToHex(r) + componentToHex(g) + componentToHex(b);
    }

    function getPathColor(index:number):string {
        try {
            let obj = document.getElementById("new_logo");
            let svg = obj["contentDocument"].getElementsByTagName("svg")[0];
            let path = svg.getElementsByTagName("path")[index];
            let color = path.style.fill.replace("rgb(", "").replace(")", "").split(",").map((x:string) => parseInt(x));
            color = rgbToHex(color[0], color[1], color[2]);
            return color;
        } catch (e) {
            console.log(e);
        }
        return DEFAULT_COLORS[index];
    }

    function colorChange(color:string, index:number):void {
        try {
            let obj = document.getElementById("new_logo");
            let svg = obj["contentDocument"].getElementsByTagName("svg")[0];

            let path = svg.getElementsByTagName("path")[index];
            path.style.fill = color;
        } catch (e) {
            console.log(e);
        }
    }

    async function saveLogo():Promise<void> {
        let obj = document.getElementById("new_logo");
        let svg = obj["contentDocument"].getElementsByTagName("svg")[0];
        let data:string = svg.outerHTML;

        let path = await resourceDir() + "logo.svg";
        fs.writeFile(path, data).then(() => {
            console.log("Saved logo");
        });
    }

    $: loaded ? colorChange(hex, selected_path) : null;
    $: loaded ? hex = getPathColor(selected_path) : null;
</script>

<img alt="Logo" src="src/base_tribar.svg" />
<object title="new_logo" id="new_logo" type="image/svg+xml" data="src/base_tribar.svg"></object>
<form on:submit|preventDefault>
    <select bind:value={selected_path}>
        <option value={0} default={true} >Color 1</option>
        <option value={1}>Color 2</option>
        <option value={2}>Color 3</option>
    </select>
</form>
<button on:click={() => hex = DEFAULT_COLORS[selected_path]}>Reset</button>
<button on:click={() => saveLogo()}>Save Logo</button>
<ColorPicker bind:hex label="Select color" isAlpha={false} toRight={true} isInput={false} isOpen={true} />
