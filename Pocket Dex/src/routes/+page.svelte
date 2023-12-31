<script>
	import { onMount } from 'svelte';
	import PokeModal from './modal.svelte';

	/**
	 * @type {any[]}
	 */
	let pokemonList = [];
	let loading = false;
	let searchTerm = '';
	let selectedType = 'all'; // Default selection is "all"
	/**
	 * @type {any}
	 */
	let selectedPokemon;
	let showModal = false;

	const baseUrl = 'https://lythical1.github.io/public/api/';
	// Currently, not all data is being utilized, mostly "All_Pokemon_Data.json".

	async function fetchPokemonList() {
		loading = true;

		try {
			const [
				pokemonResponse,
				contestEffectResponse,
				eggGroupResponse,
				encounterMethodResponse,
				growthRateResponse,
				locationResponse,
				movesResponse,
				shapeResponse,
				typesResponse,
				natureResponse
			] = await Promise.all([
				fetch(`${baseUrl}all_pokemon_data.json`),
				fetch(`${baseUrl}all_contest_effect_data.json`),
				fetch(`${baseUrl}all_egg_group_data.json`),
				fetch(`${baseUrl}all_encounter_method_data.json`),
				fetch(`${baseUrl}all_growth_rate_data.json`),
				fetch(`${baseUrl}all_location_data.json`),
				fetch(`${baseUrl}all_moves_data.json`),
				fetch(`${baseUrl}all_shape_data.json`),
				fetch(`${baseUrl}all_types_data.json`),
				fetch(`${baseUrl}natures.json`)
			]);

			const [
				pokemonData,
				contestEffectData,
				eggGroupData,
				encounterMethodData,
				growthRateData,
				locationData,
				movesData,
				shapeData,
				typesData,
				natureData
			] = await Promise.all([
				pokemonResponse.json(),
				contestEffectResponse.json(),
				eggGroupResponse.json(),
				encounterMethodResponse.json(),
				growthRateResponse.json(),
				locationResponse.json(),
				movesResponse.json(),
				shapeResponse.json(),
				typesResponse.json(),
				natureResponse.json()
			]);
			const pokemonArray = Object.values(pokemonData);

			const promises = pokemonArray.map(async (pokemon) => {
				const spriteResponse = await fetch(`https://pokeapi.co/api/v2/pokemon-form/${pokemon.id}`);
				const spriteData = await spriteResponse.json();

				return {
					name: pokemon.name,
					types: pokemon.types,
					number: pokemon.id,
					moves: pokemon.moves,
					baseExperience: pokemon.base_experience,
					sprite: spriteData.sprites.front_default
				};
			});

			pokemonList = await Promise.all(promises);
		} catch (error) {
			console.error('Error fetching data:', error);
		} finally {
			loading = false;
		}
	}

	onMount(() => {
		fetchPokemonList();
	});

	// @ts-ignore
	function showPokemonDetails(pokemon) {
		selectedPokemon = pokemon;
		showModal = true;
	}

	$: filteredPokemon = [...pokemonList].filter((pokemon) => {
		const lowerCaseName = pokemon.name.toLowerCase();
		const lowerCaseSearchTerm = searchTerm.toLowerCase();
		const isTypeMatch = selectedType === 'all' || pokemon.types.includes(selectedType);

		return (
			(lowerCaseSearchTerm === '' || lowerCaseName.includes(lowerCaseSearchTerm)) && isTypeMatch
		);
	});

	// @ts-ignore
	function getTypeColor(type) {
		const typeColors = {
			normal: '#A8A77A',
			fire: '#EE8130',
			water: '#6390F0',
			electric: '#F7D02C',
			grass: '#7AC74C',
			ice: '#96D9D6',
			fighting: '#C22E28',
			poison: '#A33EA1',
			ground: '#E2BF65',
			flying: '#A98FF3',
			psychic: '#F95587',
			bug: '#A6B91A',
			rock: '#B6A136',
			ghost: '#735797',
			dark: '#705746',
			steel: '#B7B7CE',
			fairy: '#D685AD'
		};

		// @ts-ignore
		return typeColors[type] || '#CCCCCC';
	}
</script>

<main>
    <nav class="flex-no-wrap top-0 flex flex-col md:flex-row w-full items-center justify-between z-10 py-2 shadow-md shadow-lg bg-rose-600 sticky p-4">
        <a href="../" class="text-3xl font-bold">
            <span class="text-white">The Poké Encyclopedia</span>
        </a>
        <div class="flex mt-4 md:mt-0">
            <input
                bind:value={searchTerm}
                on:input={() => {}}
                class="md:border-2 border-rose-400 bg-slate-200 h-10 pl-2 pr-8 rounded-lg text-sm focus:outline-none focus:border-rose-600"
                type="search"
                name="search"
                placeholder="Search"
            />
            <select
                bind:value={selectedType}
                class="md:ml-2 md:border-2 border-rose-400 rounded-lg text-sm focus:outline-none focus:border-rose-600 mt-2 md:mt-0"
            >
				<option value="all">All Types</option>
				<option value="normal">Normal</option>
				<option value="fire">Fire</option>
				<option value="water">Water</option>
				<option value="fighting">Fighting</option>
				<option value="grass">Grass</option>
				<option value="electric">Electirc</option>
				<option value="ice">Ice</option>
				<option value="poison">Poison</option>
				<option value="ground">Ground</option>
				<option value="psychic">Psychic</option>
				<option value="bug">Bug</option>
				<option value="rock">Rock</option>
				<option value="fairy">Fairy</option>
				<option value="ghost">Ghost</option>
				<option value="dark">Dark</option>
				<option value="steel">Steel</option>
			</select>
		</div>
	</nav>
	<div class="flex items-center">
		<div class="mx-auto p-2">
			<article class="prose lg:prose-xl">
				<div class="flex overflow-x-auto" style="white-space: normal;">
					{#if loading}
						<p>Loading...</p>
					{:else}
						<ul class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-4">
							{#each filteredPokemon as { name, types, sprite, moves, baseExperience, number } (number)}
								<button on:click={() => showPokemonDetails({ name, types, moves, baseExperience, sprite, number })}>
									<div
										class="flex-shrink-0 p-4 bg-gray-200 shadow-md shadow-gray-500/75 rounded-lg mx-auto"
									>
										<img
											src={sprite}
											alt={name}
											class="w-auto h-auto mx-auto mb-2 rounded-lg bg-gray-300 border"
										/>
										<p class="text-xl">{number}</p>
										<p class="font-bold text-2xl">{name}</p>
										<div class="flex justify-center">
											{#each types as type}
												<span
													style={`background-color: ${getTypeColor(type)};`}
													class="rounded-full px-4 py-2 mx-1 text-white text-md"
												>
													{type}
												</span>
											{/each}
										</div>
									</div>
								</button>
							{/each}
						</ul>
					{/if}
				</div>
			</article>
		</div>
	</div>
</main>

{#if selectedPokemon}
	<PokeModal bind:pokemon={selectedPokemon} bind:showModal />
{/if}
