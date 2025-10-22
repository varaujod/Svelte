<script>
    import PokemonBanner from "./lib/components/PokemonBanner.svelte";
    import PokemonCard from "./lib/components/PokemonCard.svelte";

  let pokemons = $state([]);
  let isLoading = $state(true);
  let searchTerm = $state('');

  $effect(() => {
    const fetchPokemons = async() => {
      try {
        const pokemonUrl = "http://pokeapi.co/api/v2/pokemon?limit=151&offset=0"
        const pokemonResponse = await fetch(pokemonUrl);
        const data = await pokemonResponse.json();

        const pokemonDetailsPromises = data.results.map((pokemon) => 
        fetch(pokemon.url).then((res) => res.json()));

        const detailedPokemons = await Promise.all(pokemonDetailsPromises);
        
        pokemons = detailedPokemons;
      } catch (error) {
        console.error("Falha ao buscar os pokemons: ", error);
      } finally{
        isLoading = false;
      }
    }

    fetchPokemons();
    
  });

  let filteredPokemons = $derived(
    pokemons.filter(pokemon =>
      pokemon.name.toLowerCase().includes(searchTerm.toLowerCase())
    )
  )
</script>

<main>
  <PokemonBanner />
  <div class="inputs">
    <input type="text" placeholder="Pesquisar Pokemons..." bind:value={searchTerm}/>
  </div>
  {#if isLoading}
  <p>Carregando Pokemons...</p>
  {:else if filteredPokemons.length === 0}
  <p>Nenhum pokemon encontrado</p>
  {:else}
  <div class="pokemon-list">
    {#each filteredPokemons as pokemon (pokemon.id)}
    <PokemonCard {pokemon}/>
    {/each}
  </div>
  {/if}

</main>

<style>
  .pokemon-list{
        display: flex;
        flex-flow: row wrap;
        align-items: center;
        justify-content: center;
        gap: 16px;
        padding-bottom: 32px;
    }
    
    .inputs{
        background: transparent;
        display: flex;
        position: sticky;
        top: 0;
        gap: 16px;
        justify-content: center;
        z-index: 1;
        flex-flow: row wrap;
    }

    input, select{
        border: 2px solid grey;
        border-radius: 16px;
        padding:16px;
        margin-bottom: 16px;
        margin-top: 16px;
    }

    input{
        width: 400px;
    }
</style>