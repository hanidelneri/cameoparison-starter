<script>
  import { onMount } from "svelte";
  import Welcome from "./screens/Welcome.svelte";
  import Game from "./screens/Game.svelte";
  import { select } from "./select.js";
  import { loadImage } from "./utils";

  let state = "welcome"; // other alternatives 'playing';
  let celebritiesPromise;

  let selection;
  const start = async (e) => {
    const { celebs, lookup } = await celebritiesPromise;
    selection = select(celebs, lookup, e.detail.category.slug);
    state = "playing";
  };

  const loadCelebrities = async () => {
    const res = await fetch("https://cameo-explorer.netlify.app/celebs.json");
    const data = await res.json();

    const lookup = new Map();
    data.forEach((c) => lookup.set(c.id, c));

    const subset = new Set();
    data.forEach((c) => {
      if (c.reviews >= 50) {
        subset.add(c);
        c.similar.forEach((id) => subset.add(lookup.get(id)));
      }
    });

    return {
      celebs: Array.from(subset),
      lookup,
    };
  };

  onMount(() => {
    celebritiesPromise = loadCelebrities();
    loadImage("/icons/right.svg");
    loadImage("/icons/wrong.svg");
  });
</script>

<main>
  {#if state === "welcome"}
    <Welcome on:select={start} />
  {:else if state === "playing"}
    <Game {selection} on:restart={() => (state = "welcome")} />
  {/if}
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    margin: 0 auto;
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    max-width: 800px;
  }
</style>
