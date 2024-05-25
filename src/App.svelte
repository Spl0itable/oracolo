<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { getConfig } from './config';
  import Home from './Home.svelte';
  import Note from './Note.svelte';
  import { SimplePool } from 'nostr-tools/pool';

  let currentHash = '';
  let profile: import('nostr-tools').Event;
  let name = '';
  let picture = '';
  let setPublicKey;
  let setRelays;

  const handleHashChange = () => {
    const newHash = window.location.hash.substr(1); // Remove the leading #
    if (newHash !== currentHash) {
      currentHash = newHash;
    }
  };

  onMount(async () => {
    const { publicKey, relays } = getConfig();
    setPublicKey = publicKey
    setRelays = relays

    handleHashChange();
    window.addEventListener('hashchange', handleHashChange);

    const pool = new SimplePool()
    let subscription = pool.subscribeMany(
      relays,
      [
        {
          kinds: [0],
          authors: [publicKey],
          limit: 1,
        }
      ],
      {
        onevent(event) {
          profile = event
          const parsedContent = JSON.parse(profile.content);
          name = parsedContent.name || null;
          picture = parsedContent.picture || null;
        },
        oneose() {
          subscription.close();
        }
      }
    );

  });

  onDestroy(() => {
    window.removeEventListener('hashchange', handleHashChange);
  });
</script>

{#if setPublicKey === '' }
  <div class="unfinished-setup">
    <h1>Oracolo</h1>
    <h2>Missing config!</h2>
    You need to set (at least) the <strong>author meta tag</strong> by updating this html file! Open it with an editor, look at the first lines and personalize them.
  </div>
{/if}

{#if profile && Object.keys(profile).length > 0}
  {#if currentHash === ''}
      <Home {profile} />
  {:else}
      <Note id={currentHash} {profile} />
  {/if}
{/if}

<div class="footer">
  <p>&copy; <script>document.write(new Date().getFullYear()); </script>&nbsp;All rights reserved • Follow on <a href="https://njump.me/npub16jdfqgazrkapk0yrqm9rdxlnys7ck39c7zmdzxtxqlmmpxg04r0sd733sv" style="text-decoration:underline;">Nostr</a></p><br/><br/>This blog is powered by <a href="https://njump.me">Nostr</a> and based on the <a href="https://github.com/dtonon/oracolo">Oracolo</a> framework<br/><br/>
  {#if setRelays }
    This site connects to these Nostr relays to retrieve blog posts: {setRelays.join(', ')}
  {/if}
  <script>
    window.addEventListener('load', function() {
      const currentURL = window.location.href;
      const anchorElement = document.querySelector('zap-threads');
      anchorElement.setAttribute('anchor', currentURL);
    });
  </script>
  <script type="text/javascript" src="https://unpkg.com/zapthreads/dist/zapthreads.iife.js"></script>
</div>