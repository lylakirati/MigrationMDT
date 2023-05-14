<script>
	import MotivesDotsScrolling from './../components/MotivesDotsScrolling.svelte';
    import IncomeSufficiencyScrolling from '../components/IncomeSufficiencyScrolling.svelte';
    import Budget from '../components/Budget.svelte';
	import DotGraph from '../components/DotGraph.svelte';
	import RemitScrolling from '../components/RemitScrolling.svelte';
	import Footer from '../components/Footer.svelte';
    // import Spending from '../components/Spending.svelte';
</script>

<section id="title">
    <h1>What do Central Americans run away from?</h1> 
    <div id="title-description">
        <p class='authors'>Grace Huang, Lyla Kiratiwudhikul, and Nerine Uyanik</p>
        <p><a href="https://vis-society.github.io/" target="_blank">Interactive Data Visualization & Society</a></p>
    </div>
    
    <!-- <p>
        As a note to reviewers/current viewers, since this is an MVP, there are still things that haven't been finalized yet.
        Particularly, the writing has not been completed yet and we have not finished implementing responsive generation of monetary 
        data for the final visualization. However, what's currently there should hopefully provide an idea of what our final 
        product will look like. We plan to build upon both of these items for the final version,
        as well as fixing up styling to make things more cohesive.
    </p> -->
</section>
<main>
    <section id="intro" class = "paragraph-text">
        <p>
            More than 3.8 million people from Central America have migrated to and lived in the United States as of 2019. <a href="https://www.migrationpolicy.org/article/central-american-immigrants-united-states#:~:text=The%20Central%20American%20diaspora%20is,with%20Honduran%20ancestry%20or%20origin">[1]</a>
            Immigrants from the Northern Triangle countries (El Salvador, Guatemala, and Honduras) take up over 86% of the number. 
            Still, these numbers do not include immigrants from irregular channels or people who were stopped at the U.S-Mexico border and deported. 
        </p>
        <p>
            People from these three countries embark on a hope-filled journey to find a better life for themselves or for the families they leave behind. 
            However, the routes they take are often dangerous and life-threatening, from ​​treacherous terrain to violence such as robbery and sexual assault. 
        </p>

        <b>So what has driven them to leave their home countries and risk the perilous passage to the U.S.? </b>
    </section>
    <section>
        <MotivesDotsScrolling />
    </section>
    <section>
        <IncomeSufficiencyScrolling />
    </section>
    <!-- <section class = "paragraph-text">
        <p>
            Migrating to find a job in another country can allow Central Americans to earn higher wage and send money home to aid their families.
        </p>
    </section> -->
    <section>
        <RemitScrolling />
    </section>
    <section class="budget">
        <Budget/>
    </section>
</main>
<Footer/>

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
		/* text-align: center; */
		font-family: 'Nunito', sans-serif;
		font-weight: 300;
		line-height: 1.5;
		/* font-size: 12px; */
		color: var(--main-color-text);
        margin:0 10em;
	}

    :global(button)  {
        font-family: 'Nunito', sans-serif;
        border: 1px solid black;
        padding: 5px 10px;
        border-radius: 1em;
        background: var(--button-background);
    }

    :global(button.active)  {
        /* filter: brightness(75%); */
        background: #ecac15;
        border: 2px solid black;
        box-shadow: 3px 3px 3px #BEBEBE;
        /* color: #FFFFFF; */
    }

    :global(button:hover) {
        filter: brightness(95%);
    }

    :global(input) {
        font-family: 'Nunito', sans-serif;
    }

    :global(h2) {
        font-family: 'Nunito', sans-serif;
        font-size: 1.1em;
    }

    #title {
        padding:2em 25em;
        background: var(--primary-color);
        color: var(--secondary-color-text);
        /* text-align:center; */

    }

    #title h1 {
        /* margin:0 10em; */
        font-size: 50px;
        text-align: left;
    }

    #intro {
        padding: 2em 15em;
    }

    /* .paragraph-text {
        padding: 2em 15em;
    } */

    h1 {
        font-weight: 700;
        font-size: 70px;
    }
    
    .authors {
        
    }

    :global(.step){
        display:flex;
        flex-direction:column;
        justify-content:center;
        text-align:left;
        margin: 0 auto 2rem auto;
        border-left: 2px;
        /* border-left: 2px solid var(--primary-color); */
    }


    :global(.step:last-child) {
        margin-bottom: 20em;
    }

    /* :global(.step.is-active ){
        background-color: goldenrod;
        color: #3b3b3b;
    } */

    :global(.step p) {
        padding: 1rem;
    }

    :global(.selections) {
		/* display:flex; */
        display:inline-flex;
		flex-direction:row;
		justify-content:center;
        align-items:center;
		gap: 1em;
		padding: 10px;
	}

    #title-description {
        text-align:center;
    }
    
    :global(a) {
        color: #6297D5;
    }

    :global(a:hover) {
        color: #9C6BDB;
        transition: color 0.5s ease;
        opacity:.85;
    }

</style>
