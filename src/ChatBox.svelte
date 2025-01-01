<script>
    import { onMount } from "svelte";
    
  const { message, sender } = $props();

  let messageClass = $state('');
  let avatar = $state('https://api.dicebear.com/9.x/adventurer-neutral/svg?seed=seed');
  let time = $state(new Date());

  onMount(() => {
    $effect.pre(() => {
      messageClass = message.who == sender ? 'sent' : 'recieved';
      avatar = `https://api.dicebear.com/9.x/adventurer-neutral/svg?seed=${message.who}`;
      time = new Date(message.when);
    })
  });

</script>

<div class={`message ${messageClass}`}>
  <img src={avatar} alt="avatar" />
  <div>
    <div class="message-text">
      <div class="message-owner">
        <span>{message.who}</span>
      </div>
      <p>{message.what}</p>
      <time>{time.toLocaleDateString()} -- {time.toLocaleTimeString()}</time>
    </div>
  </div>
</div>