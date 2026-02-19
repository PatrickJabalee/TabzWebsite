<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  export let person;

  let expanded = true;

  $: activeTotal = person.tabs.filter(t => !t.settled).reduce((s, t) => s + t.amount, 0);
  $: settledTotal = person.tabs.filter(t => t.settled).reduce((s, t) => s + t.amount, 0);

  function formatDate(iso) {
    return new Date(iso).toLocaleDateString('en-US', { month: 'short', day: 'numeric' });
  }
</script>

<div class="bg-white/[0.04] border border-white/[0.08] rounded-2xl overflow-hidden transition-all hover:border-white/[0.14]">
  <!-- Person header -->
  <div class="flex items-center justify-between px-5 py-4">
    <button class="flex items-center gap-3 flex-1 text-left" on:click={() => expanded = !expanded}>
      <div class="w-9 h-9 rounded-full bg-[#c8f06e]/20 text-[#c8f06e] flex items-center justify-center font-['DM_Mono',monospace] font-bold text-sm uppercase">
        {person.name[0]}
      </div>
      <div>
        <p class="font-medium text-[#f0ece3]">{person.name}</p>
        <p class="text-xs text-white/30 font-['DM_Mono',monospace]">{person.tabs.length} tab{person.tabs.length !== 1 ? 's' : ''}</p>
      </div>
    </button>
    <div class="flex items-center gap-3">
      <div class="text-right">
        <p class="text-lg font-['DM_Mono',monospace] {activeTotal > 0 ? 'text-[#c8f06e]' : 'text-white/20'}">${activeTotal.toFixed(2)}</p>
        {#if settledTotal > 0}
          <p class="text-xs text-white/20 font-['DM_Mono',monospace] line-through">${settledTotal.toFixed(2)}</p>
        {/if}
      </div>
      <button
        on:click={() => dispatch('addTab')}
        class="w-7 h-7 rounded-full bg-white/10 hover:bg-[#c8f06e]/20 hover:text-[#c8f06e] flex items-center justify-center transition-colors text-white/50"
        title="Add tab"
      >+</button>
      <button
        on:click={() => dispatch('deletePerson')}
        class="w-7 h-7 rounded-full bg-white/10 hover:bg-red-500/20 hover:text-red-400 flex items-center justify-center transition-colors text-white/30 text-xs"
        title="Remove person"
      >✕</button>
    </div>
  </div>

  <!-- Tabs list -->
  {#if expanded && person.tabs.length > 0}
    <div class="border-t border-white/[0.06] divide-y divide-white/[0.04]">
      {#each person.tabs as tab (tab.id)}
        <div class="flex items-center justify-between px-5 py-3 {tab.settled ? 'opacity-40' : ''}">
          <div class="flex items-center gap-3">
            <button
              on:click={() => dispatch('settleTab', { tabId: tab.id, settled: tab.settled })}
              class="w-5 h-5 rounded-full border transition-all flex items-center justify-center text-xs
                {tab.settled
                  ? 'bg-[#c8f06e] border-[#c8f06e] text-black'
                  : 'border-white/20 hover:border-[#c8f06e]/60'}"
            >
              {#if tab.settled}✓{/if}
            </button>
            <div>
              <p class="text-sm {tab.settled ? 'line-through' : ''}">{tab.description}</p>
              <p class="text-xs text-white/30 font-['DM_Mono',monospace]">
                {#if tab.settled && tab.settled_at}
                  ✓ settled {formatDate(tab.settled_at)}
                {:else}
                  added {formatDate(tab.created_at)}
                {/if}
              </p>
            </div>
          </div>
          <div class="flex items-center gap-3">
            <span class="font-['DM_Mono',monospace] text-sm">${tab.amount.toFixed(2)}</span>
            <button
              on:click={() => dispatch('deleteTab', tab.id)}
              class="text-white/20 hover:text-red-400 transition-colors text-xs"
              title="Delete tab"
            >✕</button>
          </div>
        </div>
      {/each}
    </div>
  {:else if expanded && person.tabs.length === 0}
    <div class="border-t border-white/[0.06] px-5 py-4 text-sm text-white/20 flex items-center gap-2">
      <span>No tabs yet</span>
      <button on:click={() => dispatch('addTab')} class="text-[#c8f06e]/60 hover:text-[#c8f06e] transition-colors">+ Add one</button>
    </div>
  {/if}
</div>
