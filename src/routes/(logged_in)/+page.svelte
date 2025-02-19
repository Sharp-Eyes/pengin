<script lang="ts">
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import data from '$lib/data';
  import messages from '$lib/ws';
  import type { Message } from '$lib/types/message';
  import { tick } from 'svelte';
  import { browser } from '$app/environment';
  import MessageInput from './MessageInput.svelte';
  import MessageComponent from './Message.svelte';
  import Markdown from '$lib/components/Markdown.svelte';

  interface UiMessage {
    message: Message;
    showAuthor: boolean;
    index: number;
  }

  let lastAuthor: string | null = null;
  let messagesUList: HTMLUListElement;
  let value = '';
  let input: HTMLTextAreaElement;
  let uiMessages: Array<UiMessage> = [];

  onMount(() => {
    messagesUList.scroll(0, 1);
    input.focus();
  });

  const checkAuthor = (author: string, index: number): boolean => {
    const res = author !== lastAuthor;
    lastAuthor = author;
    if (index == 0) {
      return true;
    }
    return res;
  };

  const mapMessages = (messages: Array<Message>) => {
    let newMessages: Array<UiMessage> = [];
    messages.forEach((m, i) =>
      newMessages.push({
        message: m,
        showAuthor: checkAuthor(m.author, i),
        index: i
      })
    );
    return newMessages;
  };

  $: {
    uiMessages = mapMessages($messages);
    if (browser)
      tick().then(() => {
        if (
          messagesUList.scrollHeight - messagesUList.offsetHeight - messagesUList.scrollTop <
          window.outerHeight / 4
        )
          messagesUList.scroll(0, messagesUList.scrollHeight);
      });
  }

  const logOut = () => {
    data.set(null);
    goto('/login');
  };

  const onSubmit = async () => {
    if (value.trim())
      fetch($data?.instanceURL + '/messages', {
        method: 'POST',
        body: JSON.stringify({ author: $data?.name, content: value }) // data?.name is fine here
      }).then(() => messagesUList.scroll(0, messagesUList.scrollHeight));
    value = '';
    await tick();
    input.style.height = '1px';
    input.style.height = `${Math.min(Math.max(26, input.scrollHeight), window.outerHeight / 3)}px`;
    input.focus(); // for mobiles
  };
</script>

<div class="message-channel">
  <div id="options-div">
    <div id="instance-info">
      <span id="instance-name">{$data?.instanceInfo?.instance_name ?? 'Pengin - loading'}</span>
      {#if $data?.instanceInfo?.description}
        <span id="instance-description">{$data.instanceInfo.description}</span>
        <span id="instance-markdown">
          <Markdown content={$data.instanceInfo.description} />
        </span>
      {/if}
    </div>
    <a id="settings-link" href="/settings"> Settings </a>
    <button id="logout-button" on:click={logOut}> Logout </button>
  </div>
  <ul bind:this={messagesUList} id="messages">
    {#each uiMessages as { message, showAuthor, index } (index)}
      <MessageComponent {message} {showAuthor} />
    {/each}
  </ul>
  <form id="message-input-form" on:submit|preventDefault={onSubmit}>
    <MessageInput bind:input bind:value on:submit={onSubmit} />
    <button id="send-button">Send</button>
  </form>
</div>

<style>
  .message-channel {
    width: 100vw;
    height: 100vh;
    display: flex;
    flex-direction: column;
  }

  #messages {
    flex-grow: 1;
    padding: 10px;
    overflow-y: auto;
    padding: 0;
  }

  #message-input-form {
    width: calc(100% - 10px);
    display: flex;
    background-color: var(--gray-200);
    color: var(--gray-600);
    padding: 2px;
    margin: 10px 5px;
    font-size: 18px;
    min-height: 41px;
    border-radius: 10px;
  }

  #send-button {
    background: none;
    color: inherit;
    border: none;
    cursor: pointer;
    margin: 3px 5px 3px 0;
    font-size: inherit;
    transition: color ease-in-out 125ms;
    cursor: pointer;
  }

  #send-button:hover {
    color: var(--gray-500);
  }

  #options-div {
    display: flex;
    background-color: var(--gray-200);
    padding: 10px;
    gap: 20px;
  }

  #instance-info {
    display: flex;
    flex-grow: 1;
    overflow: hidden;
  }

  #instance-name {
    margin-right: 15px;
    font-size: 24px;
    align-self: baseline;
  }

  #instance-description {
    font-weight: 300;
    align-self: baseline;
    white-space: nowrap;
  }

  #instance-markdown {
    display: none;
    position: absolute;
    top: 35px;
    left: 0;
    background-color: var(--gray-300);
    margin: 20px;
    padding: 10px;
    border-radius: 10px;
    max-width: 90vw;
  }

  #instance-info:hover > #instance-markdown {
    display: inline;
  }

  #settings-link {
    text-decoration: none;
    align-self: center;
    border: unset;
  }

  #logout-button {
    padding: 5px 10px;
    outline: none;
    border: unset;
    border-radius: 25px;
    background-color: var(--pink-500);
    color: var(--purple-100);
    box-shadow: 0 2px 4px var(--purple-200);
    transition: box-shadow ease-in-out 200ms, color ease-in-out 200ms,
      background-color ease-in-out 200ms;
    cursor: pointer;
  }

  #logout-button:hover {
    box-shadow: 0 5px 20px var(--purple-200);
    background-color: var(--pink-600);
  }
</style>
