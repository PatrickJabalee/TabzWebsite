<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  export let loading = false;
  export let error = '';

  let name = '';

  function submit() {
    if (!name.trim()) return;
    dispatch('login', name.trim());
  }

  function handleKey(e) {
    if (e.key === 'Enter') submit();
  }
</script>

<div class="min-h-screen bg-[#0e0e10] flex items-center justify-center px-4">
  <div class="w-full max-w-sm">
    <!-- Logo -->
    <div class="text-center mb-10">
      <p class="text-5xl mb-4">🧾</p>
      <h1 class="text-4xl font-['Instrument_Serif',serif] text-[#f0ece3]">Tabz</h1>
      <p class="text-white/30 text-sm mt-2">Track what people owe you</p>
    </div>

    <!-- Card -->
    <div class="bg-white/[0.04] border border-white/10 rounded-2xl p-6">
      <h2 class="text-lg font-['Instrument_Serif',serif] text-[#f0ece3] mb-1">Who are you?</h2>
      <p class="text-sm text-white/30 mb-5">Enter your name to access your tabs</p>

      <input
        type="text"
        bind:value={name}
        on:keydown={handleKey}
        placeholder="Your name"
        autofocus
        disabled={loading}
        class="w-full bg-white/5 border border-white/10 rounded-xl px-4 py-3 text-sm outline-none
          focus:border-[#c8f06e]/50 placeholder-white/20 transition-colors mb-4
          disabled:opacity-40 text-[#f0ece3]"
      />

      {#if error}
        <p class="text-red-400 text-xs mb-3">{error}</p>
      {/if}

      <button
        on:click={submit}
        disabled={!name.trim() || loading}
        class="w-full py-3 rounded-xl bg-[#c8f06e] text-black text-sm font-medium
          hover:bg-[#d9ff7a] disabled:opacity-30 disabled:cursor-not-allowed transition-colors"
      >
        {loading ? 'Loading...' : 'Enter'}
      </button>
    </div>

    <p class="text-center text-white/20 text-xs mt-6">
      New here? Just enter your name and we'll create your account.
    </p>
  </div>
</div>
