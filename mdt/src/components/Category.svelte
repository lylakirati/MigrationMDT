<!-- An Individual Category for Budgeting Interactive Visualization  -->

<script>
    export let title;
    export let dollars;
    export let maxValue;
    // export let percentage;
    let input;
</script>
<div>
    <h3>{title}</h3>
    <input bind:this={input} type="number" min="0" step="1" bind:value={dollars} max={`${maxValue}`}> $
    <!-- <input type="number" min="0" max="100" step="1" bind:value={percentage}> % -->
</div>

<style>
    input[type="number"] {
        width:5em;

    }

    h3 {
        margin: 0 0 1em 0;
    }
</style>