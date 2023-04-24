<script>
    import DotGraph from './DotGraph.svelte';
    import IncomeSufficiency from './IncomeSufficiency.svelte';
</script>

<main>
	<IncomeSufficiency />
</main>

<style>
    @import url("https://fonts.googleapis.com/css2?family=Nunito:wght@300;400;700&display=swap");

    :root {
        --color-bg: #ffffff;
        --color-outline: #c2c2c2;
        --color-shadow: hsl(0, 0%, 0%, 0.1);
        --color-text: #3f4252;
        --color-bg-1: hsla(0, 0%, 0%, 0.2);
        --color-shadow-1: hsl(0, 0%, 96%);
    }

    main {
		text-align: center;
		font-family: 'Nunito', sans-serif;
		font-weight: 300;
		line-height: 1;
		/* font-size: 12px; */
		color: var(--color-text);
		margin-top: 100px;
	}

    :global(button)  {
    font-family: 'Nunito', sans-serif;
    border: 1px solid black;
    padding: 5px 10px;
    border-radius: 1em;
    background: #F8CE6D;
    }

    :global(button:hover) {
        filter: brightness(95%);
    }


</style>
