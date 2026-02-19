<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  let name = '';

  function submit() {
    if (!name.trim()) return;
    dispatch('add', name.trim());
    name = '';
  }

  function handleKey(e) {
    if (e.key === 'Enter') submit();
    if (e.key === 'Escape') dispatch('close');
  }
</script>

<div
  class="fixed inset-0 bg-black/60 backdrop-blur-sm z-50 flex items-end sm:items-center justify-center p-4"
  on:click|self={() => dispatch('close')}
  on:keydown={handleKey}
  role="dialog"
  aria-modal="true"
>
  <div class="bg-[#1a1a1e] border border-white/10 rounded-2xl p-6 w-full max-w-sm shadow-2xl">
    <h2 class="text-lg mb-1 font-['Instrument_Serif',serif]">Who owes you?</h2>
    <p class="text-sm text-white/30 mb-5">Add a person to start tracking their tab</p>

    <input
      type="text"
      bind:value={name}
      on:keydown={handleKey}
      placeholder="Name"
      autofocus
      class="w-full bg-white/5 border border-white/10 rounded-xl px-4 py-3 text-sm outline-none focus:border-[#c8f06e]/50 placeholder-white/20 transition-colors mb-4"
    />

    <div class="flex gap-2">
      <button
        on:click={() => dispatch('close')}
        class="flex-1 py-3 rounded-xl bg-white/5 hover:bg-white/10 text-sm transition-colors text-white/50"
      >Cancel</button>
      <button
        on:click={submit}
        disabled={!name.trim()}
        class="flex-1 py-3 rounded-xl bg-[#c8f06e] text-black text-sm font-medium hover:bg-[#d9ff7a] disabled:opacity-30 disabled:cursor-not-allowed transition-colors"
      >Add Person</button>
    </div>
  </div>
</div>
