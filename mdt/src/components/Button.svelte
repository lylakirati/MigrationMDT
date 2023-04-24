<!-- A custom button that changes colors when clicked! Though not used right now -->
<script>
	export let content; // content of button
    export let onClick; // function to call on Click, along with changing style
	console.log({ content });
    export let active = false;
    

</script>

<button class={active ? "pressed" : ""} on:click = {() => {active = !active; onClick();}}>
    {content}
</button>

<style>
button {
    font-family: 'Nunito', sans-serif;
    border: 1px solid black;
    padding: 5px 10px;
    border-radius: 1em;
    background: #F8CE6D;
}

button:hover {
    filter: brightness(95%);
}

.pressed {
    background-color: #DDB6FC;
}

</style>
