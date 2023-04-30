<script>
    import Scroller from "@sveltejs/svelte-scroller";
    import DotGraph from './DotGraph.svelte';
    import Spending from './Spending.svelte';
	import IncomeSufficiency from "./IncomeSufficiency.svelte";

    let count, index, offset, progress;
    let width, height;
</script>

<Scroller
    top={0.0} 
    bottom={1}
    threshold={0.5}
    bind:count
    bind:index
    bind:offset
    bind:progress
>

    <div 
        class="background" 
        slot="background"
        bind:clientWidth={width}
        bind:clientHeight={height}
    >
        <div class="progress-bars">
        <!-- populated the background container with 3 progress bars to visualize the bounded variables of the scroller -->
        <!-- <p>current section: <strong>{index + 1}/{count}</strong></p>
        <progress value={count ? (index + 1) / count : 0} />

        <p>offset in current section</p>
        <progress value={offset || 0} />

        <p>total progress</p>
        <progress value={progress || 0} /> -->
        </div>
    </div>

    <div class="foreground" slot="foreground">
        <section>
            <h1>What do Central Americans run away from?</h1>

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
            Poverty, food insecurity, and lack of economic opportunities for sustainable livelihoods have been cited as the top factors behind migration. 
            According to the survey conducted by the World Food Programme in 2021, more than 80 percent of households reported that they want to emigrate in search of employment opportunities. 
            A large number of people also lack money to secure food and other basic needs, which motivates them to find jobs in other countries and send money home. 
            <IncomeSufficiency />
        </section>
        <section><DotGraph /></section>
        <section>This is the third section.</section>
        <section>
            <Spending />
        </section>
        <section>This is the fifth section.</section>
        <section>This is the sixth section.</section>
    </div>
</Scroller>


<style>
    .background {
        width: 100%;
        height: 100vh;
        position: relative;
        outline: green solid 3px;
    }
  
    .foreground {
        width: 80%;
        margin: 0 auto;
        height: auto;
        position: relative;
        outline: red solid 3px;
    }
  
    .progress-bars {
        position: absolute;
        background: rgba(170, 51, 120, 0.2) /*  40% opaque */;
        visibility: visible;
    }
  
    section {
        height: 200vh;
        background-color: rgba(0, 0, 0, 0.2); /* 20% opaque */
        /* color: white; */
        /* outline: magenta solid 3px; */
        text-align: center;
        /* max-width: 1000px; /* adjust at will  */
        color: black;
        padding: 1em;
        margin: 0 0 2em 0;
    }
</style>
  