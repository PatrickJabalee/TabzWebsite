<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  export let person;

  let description = '';
  let amount = '';

  function submit() {
    if (!description.trim() || !amount || isNaN(parseFloat(amount))) return;
    dispatch('add', { description: description.trim(), amount });
    description = '';
    amount = '';
  }

  function handleKey(e) {
    if (e.key === 'Enter') submit();
    if (e.key === 'Escape') dispatch('close');
  }

  const suggestions = ['Dinner', 'Drinks', 'Uber', 'Groceries', 'Rent', 'Concert'];
</script>

<div
  class="fixed inset-0 bg-black/60 backdrop-blur-sm z-50 flex items-end sm:items-center justify-center p-4"
  on:click|self={() => dispatch('close')}
  role="dialog"
  aria-modal="true"
>
  <div class="bg-[#1a1a1e] border border-white/10 rounded-2xl p-6 w-full max-w-sm shadow-2xl">
    <h2 class="text-lg mb-1 font-['Instrument_Serif',serif]">Add a tab</h2>
    <p class="text-sm text-white/30 mb-5">
      What does <span class="text-[#c8f06e]">{person.name}</span> owe you for?
    </p>

    <!-- Quick suggestions -->
    <div class="flex flex-wrap gap-2 mb-4">
      {#each suggestions as s}
        <button
          on:click={() => description = s}
          class="px-3 py-1 rounded-full text-xs border transition-colors
            {description === s
              ? 'border-[#c8f06e] text-[#c8f06e] bg-[#c8f06e]/10'
              : 'border-white/10 text-white/30 hover:border-white/30 hover:text-white/60'}"
        >{s}</button>
      {/each}
    </div>

    <input
      type="text"
      bind:value={description}
      on:keydown={handleKey}
      placeholder="Description"
      autofocus
      class="w-full bg-white/5 border border-white/10 rounded-xl px-4 py-3 text-sm outline-none focus:border-[#c8f06e]/50 placeholder-white/20 transition-colors mb-3"
    />

    <div class="relative mb-4">
      <span class="absolute left-4 top-1/2 -translate-y-1/2 text-white/30 font-['DM_Mono',monospace]">$</span>
      <input
        type="number"
        bind:value={amount}
        on:keydown={handleKey}
        placeholder="0.00"
        min="0"
        step="0.01"
        class="w-full bg-white/5 border border-white/10 rounded-xl pl-8 pr-4 py-3 text-sm outline-none focus:border-[#c8f06e]/50 placeholder-white/20 transition-colors font-['DM_Mono',monospace]"
      />
    </div>

    <div class="flex gap-2">
      <button
        on:click={() => dispatch('close')}
        class="flex-1 py-3 rounded-xl bg-white/5 hover:bg-white/10 text-sm transition-colors text-white/50"
      >Cancel</button>
      <button
        on:click={submit}
        disabled={!description.trim() || !amount}
        class="flex-1 py-3 rounded-xl bg-[#c8f06e] text-black text-sm font-medium hover:bg-[#d9ff7a] disabled:opacity-30 disabled:cursor-not-allowed transition-colors"
      >Add Tab</button>
    </div>
  </div>
</div>
