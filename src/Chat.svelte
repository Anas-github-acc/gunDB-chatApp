<script>
  import Login from './Login.svelte';
  import ChatBox from './ChatBox.svelte';
  import { username, loggedIn, user } from './user';
  import { onMount } from 'svelte';
  import debounce from 'lodash.debounce';

  import GUN from 'gun';
  import SEA from 'gun/sea';

  // const db = new GUN('https://gunjs-chat.herokuapp.com/gun');
  const db = GUN();

  let newMsg = $state('');
  let messages = $state([]);

  let hello = $state(0);


  let scrollBottom = $state(null);
  let lastScrollTop ;
  let canAutoScroll = $state(true);
  let unreadMessages = $state(false);

  function autoScroll() {
    setTimeout(() => scrollBottom?.scrollIntoView({ behavior: 'auto' }), 50);
    unreadMessages = false;
  }

  function watchScroll(e) {
    canAutoScroll = (e.target.scrollTop || Infinity) > lastScrollTop;
    lastScrollTop = e.target.scrollTop;
  }

  let debouncedWatchScroll = debounce(watchScroll, 1000);

  // $effect(() => {
  //   debouncedWatchScroll = debounce(watchScroll, 1000);
  // });

  onMount(async () => {
    var match = {
      // lexical queries are kind of like a limited RegEx or Glob.
      '.': {
        // property selector
        '>': new Date(+new Date() - 1 * 1000 * 60 * 60 * 10).toISOString(), // find any indexed property larger ~3 hours ago
      },
      '-': 1, // filter in reverse
    };

    db.get('chat')
      .map(match)
      .once(async function(data, id) {
        // console.log('data - ', data);
        if(data) {
          const key = '123#';
          
          var message = {
            who: await db.user(data).get('alias'), // a user might lie who they are! So let the user system detect whose data it is
            what: await SEA.decrypt(data.what, key) + '', // force decryption to string
            when: GUN.state.is(data, 'what'), // we can use GUN's state to detect metadata
          }
          
          // console.log('message - ', message);
          // console.log('message - ');
          // console.log(message);

          if(message.what) {
            messages = [...messages.slice(-100), message].sort((a, b) => a.when - b.when);
            if (canAutoScroll) {
              autoScroll();
            } else {
              unreadMessages = true;
            }
          }
        }
      }
    );
  });

  async function sendMessage(event) {
    event.preventDefault()
    const encrypted = await SEA.encrypt(newMsg, '123#');

    const message = await user.get('all').set({ what: encrypted});

    const index = new Date().toISOString();
    // db.get('chat').set({ [index]: message });
    db.get('chat').get(index).put(message);

    console.log('message - ', index);

    // const emsg = await db.get('chat').get(message).get('what').then();
    // const msgd = await SEA.decrypt(emsg, '$123');
    // console.log(msgd);

    hello = hello + 1;

    newMsg = '';
    canAutoScroll = true;
    autoScroll();
  }

</script>

<div class="container">
  {#if $loggedIn}
    <main onscroll="{debouncedWatchScroll}">
      {#each messages as message } <!--we are not using any key here -->
        <ChatBox {message} sender={$username} />
      {/each}

      
      <div class="dummy" bind:this={scrollBottom}></div>
    </main>
    
    <h1>{hello}</h1>
    <form onsubmit={sendMessage}>
      <input type="text" placeholder="Type a message..." bind:value={newMsg} maxlength="100" />

      <button type="submit" disabled={!newMsg}>Send</button>
    </form>


    {#if !canAutoScroll}
    <div class="scroll-button">
      <button onclick={autoScroll} class:red={unreadMessages}>
        {#if unreadMessages}
          scroll down to see new messages
        {/if}
        👇
      </button>
    </div>
   {/if}
  {:else}
    <main>
      <Login />
    </main>
  {/if}
</div>
