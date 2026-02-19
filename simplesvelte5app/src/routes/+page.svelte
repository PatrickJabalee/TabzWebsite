<script>
  import { onMount } from 'svelte';
  import { writable, derived } from 'svelte/store';
  import { browser } from '$app/environment';
  import { supabase } from '$lib/supabase.js';
  import LoginScreen from '$lib/LoginScreen.svelte';
  import PersonCard from '$lib/PersonCard.svelte';
  import AddPersonModal from '$lib/AddPersonModal.svelte';
  import AddTabModal from '$lib/AddTabModal.svelte';

  // --- Auth state ---
  let currentUser = null;       // { id, name }
  let authLoading = true;
  let authError = '';

  // --- App state ---
  let people = writable([]);    // [{ name, tabs: [...] }]
  let dataLoading = false;
  let showAddPerson = false;
  let showAddTab = false;
  let selectedPerson = null;
  let filter = 'all';

  const totalOwed = derived(people, $people =>
    $people.reduce((sum, p) => sum + p.tabs.filter(t => !t.settled).reduce((s, t) => s + t.amount, 0), 0)
  );

  // --- On mount: check localStorage for saved user ---
  onMount(async () => {
    if (!browser) return;
    const saved = localStorage.getItem('tabz-user');
    if (saved) {
      currentUser = JSON.parse(saved);
      await loadData();
    }
    authLoading = false;
  });

  // --- Login ---
  async function handleLogin(e) {
    const name = e.detail;
    authLoading = true;
    authError = '';

    try {
      // Check if user exists
      let { data: existing } = await supabase
        .from('users')
        .select('*')
        .eq('name', name)
        .maybeSingle();

      if (existing) {
        currentUser = existing;
      } else {
        // Create new user
        const { data: created, error } = await supabase
          .from('users')
          .insert({ name })
          .select()
          .single();

        if (error) throw error;
        currentUser = created;
      }

      localStorage.setItem('tabz-user', JSON.stringify(currentUser));
      await loadData();
    } catch (err) {
      authError = 'Something went wrong. Try again.';
      console.error(err);
    } finally {
      authLoading = false;
    }
  }

  // --- Logout ---
  function logout() {
    localStorage.removeItem('tabz-user');
    currentUser = null;
    people.set([]);
  }

  // --- Load all tabs for this user ---
  async function loadData() {
    dataLoading = true;
    const { data: tabs, error } = await supabase
      .from('tabs')
      .select('*')
      .eq('owner_id', currentUser.id)
      .order('created_at', { ascending: true });

    if (error) { console.error(error); dataLoading = false; return; }

    // Group tabs by person_name
    const grouped = {};
    for (const tab of tabs) {
      if (!grouped[tab.person_name]) grouped[tab.person_name] = [];
      grouped[tab.person_name].push(tab);
    }

    people.set(
      Object.entries(grouped).map(([name, tabs]) => ({ name, tabs }))
    );
    dataLoading = false;
  }

  // --- Add person (local only until they get a tab) ---
  function addPerson(name) {
    people.update(p => {
      if (p.find(x => x.name === name)) return p;
      return [...p, { name, tabs: [] }];
    });
    showAddPerson = false;
  }

  // --- Remove person (delete all their tabs from DB) ---
  async function deletePerson(name) {
    await supabase
      .from('tabs')
      .delete()
      .eq('owner_id', currentUser.id)
      .eq('person_name', name);

    people.update(p => p.filter(x => x.name !== name));
  }

  // --- Add tab ---
  async function addTab(personName, { description, amount }) {
    const { data: tab, error } = await supabase
      .from('tabs')
      .insert({
        owner_id: currentUser.id,
        person_name: personName,
        description,
        amount: parseFloat(amount),
      })
      .select()
      .single();

    if (error) { console.error(error); return; }

    people.update(p =>
      p.map(person =>
        person.name === personName
          ? { ...person, tabs: [...person.tabs, tab] }
          : person
      )
    );

    showAddTab = false;
    selectedPerson = null;
  }

  // --- Settle / unsettle tab ---
  async function settleTab(personName, tabId, currentlySettled) {
    const newSettled = !currentlySettled;
    const { error } = await supabase
      .from('tabs')
      .update({
        settled: newSettled,
        settled_at: newSettled ? new Date().toISOString() : null,
      })
      .eq('id', tabId);

    if (error) { console.error(error); return; }

    people.update(p =>
      p.map(person =>
        person.name === personName
          ? {
              ...person,
              tabs: person.tabs.map(t =>
                t.id === tabId
                  ? { ...t, settled: newSettled, settled_at: newSettled ? new Date().toISOString() : null }
                  : t
              )
            }
          : person
      )
    );
  }

  // --- Delete tab ---
  async function deleteTab(personName, tabId) {
    await supabase.from('tabs').delete().eq('id', tabId);

    people.update(p =>
      p.map(person =>
        person.name === personName
          ? { ...person, tabs: person.tabs.filter(t => t.id !== tabId) }
          : person
      )
    );
  }

  function openAddTab(person) {
    selectedPerson = person;
    showAddTab = true;
  }

  $: filteredPeople = $people.filter(p => {
    if (filter === 'active') return p.tabs.some(t => !t.settled);
    if (filter === 'settled') return p.tabs.length > 0 && p.tabs.every(t => t.settled);
    return true;
  });
</script>

<!-- Loading splash -->
{#if authLoading}
  <div class="min-h-screen bg-[#0e0e10] flex items-center justify-center">
    <p class="text-white/20 font-['DM_Mono',monospace]">Loading...</p>
  </div>

<!-- Login screen -->
{:else if !currentUser}
  <LoginScreen error={authError} on:login={handleLogin} />

<!-- Main app -->
{:else}
  <main class="min-h-screen bg-[#0e0e10] text-[#f0ece3] font-['Instrument_Serif',serif]">
    <!-- Header -->
    <header class="sticky top-0 z-10 bg-[#0e0e10]/90 backdrop-blur-sm border-b border-white/5 px-6 py-4 flex items-center justify-between">
      <div class="flex items-center gap-3">
        <span class="text-2xl">🧾</span>
        <h1 class="text-2xl tracking-tight">Tabz</h1>
      </div>
      <div class="flex items-center gap-5">
        <div class="text-right">
          <p class="text-xs text-white/40 uppercase tracking-widest font-['DM_Mono',monospace]">Total Owed</p>
          <p class="text-xl text-[#c8f06e] font-['DM_Mono',monospace]">${$totalOwed.toFixed(2)}</p>
        </div>
        <button
          on:click={logout}
          class="text-xs text-white/20 hover:text-white/50 font-['DM_Mono',monospace] transition-colors border border-white/10 hover:border-white/20 rounded-full px-3 py-1.5"
        >
          {currentUser.name} · logout
        </button>
      </div>
    </header>

    <div class="max-w-2xl mx-auto px-4 py-8">
      <!-- Filter + Add -->
      <div class="flex items-center justify-between mb-6">
        <div class="flex gap-1 bg-white/5 rounded-full p-1">
          {#each ['all', 'active', 'settled'] as f}
            <button
              on:click={() => filter = f}
              class="px-4 py-1.5 rounded-full text-sm transition-all capitalize font-['DM_Mono',monospace]
                {filter === f ? 'bg-[#c8f06e] text-black font-medium' : 'text-white/40 hover:text-white/70'}"
            >{f}</button>
          {/each}
        </div>
        <button
          on:click={() => showAddPerson = true}
          class="flex items-center gap-2 bg-[#c8f06e] text-black px-4 py-2 rounded-full text-sm font-medium hover:bg-[#d9ff7a] transition-colors"
        >
          <span class="text-lg leading-none">+</span> Add Person
        </button>
      </div>

      <!-- Loading data -->
      {#if dataLoading}
        <div class="text-center py-24 text-white/20 font-['DM_Mono',monospace]">Loading your tabs...</div>

      <!-- Empty state -->
      {:else if filteredPeople.length === 0}
        <div class="text-center py-24 text-white/20">
          <p class="text-5xl mb-4">💸</p>
          <p class="text-lg">No tabs here</p>
          <p class="text-sm mt-1">Add someone and start tracking what they owe</p>
        </div>

      <!-- People list -->
      {:else}
        <div class="space-y-4">
          {#each filteredPeople as person (person.name)}
            <PersonCard
              {person}
              on:addTab={() => openAddTab(person)}
              on:settleTab={e => settleTab(person.name, e.detail.tabId, e.detail.settled)}
              on:deleteTab={e => deleteTab(person.name, e.detail)}
              on:deletePerson={() => deletePerson(person.name)}
            />
          {/each}
        </div>
      {/if}
    </div>
  </main>

  {#if showAddPerson}
    <AddPersonModal on:add={e => addPerson(e.detail)} on:close={() => showAddPerson = false} />
  {/if}

  {#if showAddTab && selectedPerson}
    <AddTabModal
      person={selectedPerson}
      on:add={e => addTab(selectedPerson.name, e.detail)}
      on:close={() => { showAddTab = false; selectedPerson = null; }}
    />
  {/if}
{/if}
