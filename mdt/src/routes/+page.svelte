<script>
	import MotivesDotsScrolling from './../components/MotivesDotsScrolling.svelte';
    import IncomeSufficiencyScrolling from '../components/IncomeSufficiencyScrolling.svelte';
    import Budget from '../components/Budget.svelte';
	import DotGraph from '../components/DotGraph.svelte';
	import RemitScrolling from '../components/RemitScrolling.svelte';
    // import Spending from '../components/Spending.svelte';
</script>

<section id="title">
    <h1>What do Central Americans run away from?</h1>
    <p class='authors'>Grace Huang, Lyla Kiratiwudhikul, Nerine Uyanik</p>
</section>
<main>
    <section id="intro">
        <p>
            More than 3.8 million people from Central America have migrated to and lived in the United States as of 2019. <a href="https://www.migrationpolicy.org/article/central-american-immigrants-united-states#:~:text=The%20Central%20American%20diaspora%20is,with%20Honduran%20ancestry%20or%20origin">[1]</a>
            Immigrants from the Northern Triangle countries (El Salvador, Guatemala, and Honduras) take up over 86% of the number. 
            Still, these numbers do not include irregular (unauthorized) immigrants and those who were stopped at the U.S-Mexico border and deported. 
        </p>
        <p>
            People from these countries embark on a hope-filled journey to find a better life for themselves or for the families they leave behind. 
            However, the routes they take are dangerous and life-threatening, from ​​treacherous terrain to violence such as robbery and sexual assault. 
        </p>

        <b>So what has driven them to leave their home countries and risk the perilous passage to the U.S.? </b>
    </section>
    <section>
        <MotivesDotsScrolling />
    </section>
    <section>
        <IncomeSufficiencyScrolling />
    </section>
    <section>
        <RemitScrolling />
    </section>
    <section>
        <Budget/>
    </section>
</main>

<!-- <h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p> -->

<style>
    
    @import url("https://fonts.googleapis.com/css2?family=Nunito:wght@300;400;700&display=swap");
    :global(body) {
        margin:0;
    }
    :root {
        --primary-color: #3F4952;
        --main-background-color: #FFFCF8;
        --main-color-text: black;
        --secondary-color-text: #FAEBCE;
        --button-background: #F8CE6D;
    }
    

    :global(main) {
		text-align: center;
		font-family: 'Nunito', sans-serif;
		font-weight: 300;
		line-height: 1.5;
		/* font-size: 12px; */
		color: var(--main-color-text);
        margin:0 5em;
	}

    :global(button)  {
    font-family: 'Nunito', sans-serif;
    border: 1px solid black;
    padding: 5px 10px;
    border-radius: 1em;
    background: var(--button-background);
    }

    :global(button:hover) {
        filter: brightness(95%);
    }

    :global(input) {
        font-family: 'Nunito', sans-serif;
    }

    #title {
        padding:5em;
        background: var(--primary-color);
        color: var(--secondary-color-text);
        text-align:center;

    }
    h1 {
        font-weight: 700;
        font-size: 70px;
    }
    
    .authors {
        opacity:.85;
    }

</style>
