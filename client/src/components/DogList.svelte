<script lang="ts">
    import { onMount } from "svelte";

    interface Dog {
        id: number;
        name: string;
        breed: string;
        status: string; // Assuming "available" or other statuses
    }

    export let dogs: Dog[] = [];
    let loading = true;
    let error: string | null = null;

    // Dropdown state
    let isDropdownOpen = false;
    let selectedDog: Dog | null = null;

    // Checkbox state
    let showOnlyAvailable = false;

    async function fetchDogs() {
        loading = true;
        try {
            const response = await fetch('/api/dogs');
            if(response.ok) {
                dogs = await response.json();
            } else {
                error = `Failed to fetch data: ${response.status} ${response.statusText}`;
            }
        } catch (err) {
            error = `Error: ${err instanceof Error ? err.message : String(err)}`;
        } finally {
            loading = false;
        }
    }

    onMount(() => {
        fetchDogs();
    });

    function toggleDropdown() {
        isDropdownOpen = !isDropdownOpen;
    }

    function selectDog(dog: Dog) {
        selectedDog = dog;
        isDropdownOpen = false;
        window.location.href = `/dog/${dog.id}`;
    }

    function filteredDogs() {
        return showOnlyAvailable ? dogs.filter(dog => dog.status === "available") : dogs;
    }
</script>

<div>
    <h2 class="text-2xl font-medium mb-6 text-slate-100">Available Dogs</h2>

    <!-- Checkbox to filter available dogs -->
    <div class="mb-4">
        <label class="flex items-center text-slate-100">
            <input 
                type="checkbox" 
                bind:checked={showOnlyAvailable} 
                class="mr-2 rounded border-slate-700 bg-slate-800 text-blue-500 focus:ring-blue-500"
            />
            Show only available dogs
        </label>
    </div>

    <!-- Dropdown for selecting a dog -->
    {#if !loading && !error && filteredDogs().length > 0}
        <div class="relative mb-6">
            <button 
                on:click={toggleDropdown}
                class="w-full bg-slate-800 text-left border border-slate-700 rounded-lg px-4 py-3 flex justify-between items-center hover:border-blue-500/50 transition-all duration-300"
            >
                <span class="text-slate-100">{selectedDog ? selectedDog.name : 'Select a dog'}</span>
                <svg 
                    xmlns="http://www.w3.org/2000/svg" 
                    class="h-5 w-5 text-slate-400 transition-transform duration-300" 
                    style="transform: {isDropdownOpen ? 'rotate(180deg)' : 'rotate(0deg)'}"
                    fill="none" 
                    viewBox="0 0 24 24" 
                    stroke="currentColor"
                >
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
                </svg>
            </button>

            {#if isDropdownOpen}
                <div class="absolute z-10 mt-2 w-full bg-slate-800 border border-slate-700 rounded-lg shadow-lg max-h-60 overflow-y-auto">
                    {#each filteredDogs() as dog}
                        <div 
                            on:click={() => selectDog(dog)}
                            class="px-4 py-3 hover:bg-slate-700 cursor-pointer text-slate-100 flex justify-between items-center"
                        >
                            <span>{dog.name} ({dog.breed})</span>
                        </div>
                    {/each}
                </div>
            {/if}
        </div>
    {/if}

    {#if loading}
        <!-- loading animation -->
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
            {#each Array(6) as _, i}
                <div class="bg-slate-800/60 backdrop-blur-sm rounded-xl overflow-hidden shadow-lg border border-slate-700/50">
                    <div class="p-6">
                        <div class="animate-pulse">
                            <div class="h-6 bg-slate-700 rounded w-3/4 mb-3"></div>
                            <div class="h-4 bg-slate-700 rounded w-1/2 mb-4"></div>
                            <div class="h-4 bg-slate-700 rounded w-1/4 mt-6"></div>
                        </div>
                    </div>
                </div>
            {/each}
        </div>
    {:else if error}
        <!-- error display -->
        <div class="text-center py-12 bg-slate-800/50 backdrop-blur-sm rounded-xl border border-slate-700">
            <p class="text-red-400">{error}</p>
        </div>
    {:else if filteredDogs().length === 0}
        <!-- no dogs found -->
        <div class="text-center py-12 bg-slate-800/50 backdrop-blur-sm rounded-xl border border-slate-700">
            <p class="text-slate-300">No dogs available at the moment.</p>
        </div>
    {:else}
        <!-- dog list -->
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
            {#each filteredDogs() as dog (dog.id)}
                <a 
                    href={`/dog/${dog.id}`} 
                    class="group block bg-slate-800/60 backdrop-blur-sm rounded-xl overflow-hidden shadow-lg border border-slate-700/50 hover:border-blue-500/50 hover:shadow-blue-500/10 hover:shadow-xl transition-all duration-300 hover:translate-y-[-6px]"
                >
                    <div class="p-6 relative">
                        <div class="absolute inset-0 bg-gradient-to-r from-blue-600/10 to-purple-600/5 opacity-0 group-hover:opacity-100 transition-opacity duration-300"></div>
                        <div class="relative z-10">
                            <h3 class="text-xl font-semibold text-slate-100 mb-2 group-hover:text-blue-400 transition-colors">{dog.name}</h3>
                            <p class="text-slate-400 mb-4">{dog.breed}</p>
                            <div class="mt-4 text-sm text-blue-400 font-medium flex items-center">
                                <span>View details</span>
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 ml-1 transform transition-transform duration-300 group-hover:translate-x-2" viewBox="0 0 20 20" fill="currentColor">
                                    <path fill-rule="evenodd" d="M12.293 5.293a1 1 0 011.414 0l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414-1.414L14.586 11H3a1 1 0 110-2h11.586l-2.293-2.293a1 1 0 010-1.414z" clip-rule="evenodd" />
                                </svg>
                            </div>
                        </div>
                    </div>
                </a>
            {/each}
        </div>
    {/if}
</div>