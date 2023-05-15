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
    
    <section>
        <input bind:this={input} type="number" min="0" step="1" bind:value={dollars} max={`${maxValue}`}> 
        <div>$</div>
    </section>
    <!-- <input type="number" min="0" max="100" step="1" bind:value={percentage}> % -->
</div>

<style>
    input[type="number"] {
       min-width:5em;

    }

    h3 {
        margin: 0 0 1em 0;
    }

    div {
        display:flex;
        flex-direction:column;
        align-items:center;
    }

    section {
        display:flex;
        align-items:center;
        gap:5px;
    }
</style>