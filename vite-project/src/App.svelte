<script>
  import PokemonCard from "./lib/components/PokemonCard.svelte";
  import PokemonBanner from "./lib/components/PokemonBanner.svelte";
  import PokemonDetailModal from "./lib/components/PokemonDetailModal.svelte";

  let pokemons = $state([]);
  let isLoading = $state(true);
  let searchTerm = $state('');
  let selectedType = $state(null);
  
  let selectedPokemon = $state(null);

  $effect( () =>{
      const fetchPokemons = async() =>{
          try{
              const pokemonUrl = 'https://pokeapi.co/api/v2/pokemon?limit=151&offset=0';
              const pokemonResponse = await fetch(pokemonUrl);
              const data = await pokemonResponse.json();

              const pokemonDetailsPromises = data.results.map(pokemon =>
                  fetch(pokemon.url).then(res => res.json())
              );

              const detailedPokemons = await Promise.all(pokemonDetailsPromises);
              pokemons = detailedPokemons;
          }catch(error){
              console.error("Falha ao buscar Pokémon:", error);
          }finally{
              isLoading = false;
          }
      }
      fetchPokemons();
  });

  const availableTypes = $derived(
      [...new Set(pokemons.flatMap(p => p.types.map(t => t.type.name)))].sort()
  );

  let filteredPokemons = $derived(
      pokemons
          .filter(pokemon =>
              pokemon.name.toLowerCase().includes(searchTerm.toLowerCase())
          )
          .filter(pokemon => {
              if (!selectedType) return true;
              return pokemon.types.some(t => t.type.name === selectedType);
          })
  );

  const openModal = (pokemon) => {
      selectedPokemon = pokemon;
  };

  const closeModal = () => {
      selectedPokemon = null;
  };

</script>

<main>
  <PokemonBanner></PokemonBanner>
  <div class="inputs">

      <input 
          type="text" 
          placeholder="Pesquisar Pokemon..." 
          bind:value={searchTerm}
      />

      <select bind:value={selectedType}>
          <option value={null}>Todos os tipos</option>

          {#each availableTypes as type}
              <option value={type}>{type}</option>
          {/each}
      </select>
  </div>

  
  {#if isLoading}
      <p>Carregando pokemons...</p>
  {:else if filteredPokemons.length === 0}
      <p>Nenhum Pokémon encontrado com esses filtros.</p>
  {:else}
      <div class="pokemon-list">
          {#each filteredPokemons as pokemon (pokemon.id)}
              <PokemonCard {pokemon} onSelect={() => openModal(pokemon)} />
          {/each}
      </div>
  {/if}

  {#if selectedPokemon}
      <PokemonDetailModal pokemon={selectedPokemon} onClose={closeModal} />
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