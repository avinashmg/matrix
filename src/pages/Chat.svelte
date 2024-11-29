<script>
  export let id;
  import showdown from "showdown";
  import IdCard from "../lib/IdCard.svelte";
  import { onMount } from "svelte";
  import CharList from "../lib/CharList.svelte";
  import ChatHistory from "../lib/ChatHistory.svelte";
  import { Account, Client } from "appwrite";
  let textinput = "";
  let messages = [];
  let character = {
    name: "",
    subtitle: "",
    id_image: "",
    system: "",
    self_decription: "",
  };
  let ai_role = character.name;
  let ws_url = import.meta.env.VITE_SERVER_URL
    ? `wss://${import.meta.env.VITE_SERVER_URL}`
    : `ws://localhost:3024`;

  const initial_messages = [
    {
      role: "system",
      content: character.system,
    },
    {
      role: "assistant",
      content: character.self_decription,
    },
  ];
  let typing = false;
  let streamText = "";

  const converter = new showdown.Converter();

  const chatScroll = () => {
    setTimeout(() => {
      const chatscreen = document.getElementsByClassName("chatscreen")[0];
      chatscreen.scrollTop = chatscreen.scrollHeight;
    }, 50);
  };
  onMount(() => {
    messages = JSON.parse(localStorage.getItem("chat_history")) || [];
    chatScroll();
  });

  const chatToLocalStorage = () => {
    localStorage.setItem("chat_history", JSON.stringify(messages));
  };

  const send = async (regen = false) => {
    chatScroll();
    if (typing) return;
    if (textinput === "" && !regen) return;
    if (regen !== true)
      messages = [
        ...messages,
        {
          id: Date.now().toString(),
          role: "user",
          content: textinput,
        },
      ];

    typing = true;
    streamText = "";
    ws.send(
      JSON.stringify({
        type: "send",
        id: id,
        message: textinput,
      })
    );
    textinput = "";
    chatScroll();
  };
  textinput = "";

  const enterKeyHandler = (e) => {
    if (!e.shiftKey && e.key === "Enter") {
      e.preventDefault();
      send();
    }
  };

  const regen_last = () => {
    typing = true;
    streamText = "";
    ws.send(
      JSON.stringify({
        type: "regen",
        id: id,
      })
    );
    messages = messages.slice(0, messages.length - 1);
  };

  let leftSidebarOpen = false;
  const toggleLeftSidebar = () => {
    leftSidebarOpen = !leftSidebarOpen;
  };

  let char_id = "";
  const newchat = async () => {
    if (char_id == "") return;
    messages = [];
    chatToLocalStorage();
    chatScroll();
    let server_url = import.meta.env.VITE_SERVER_URL;
    const response = await fetch(
      server_url
        ? `https://${server_url}/newchat`
        : "http://localhost:3024/newchat",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          id: char_id,
        }),
      }
    );
    const data = await response.json();
    if (data.id) {
      window.location.href = `/chat/${data.id}`;
    }
  };

  const logout = async () => {
    const client = new Client().setProject("v-friend"); // Your project ID
    const account = new Account(client);
    await account.deleteSession("current");
    window.location.reload();
  };

  // Start a websocket connection
  let ws;
  const connectWebSocket = () => {
    ws = new WebSocket(ws_url);
    ws.onopen = () => {
      ws.send(
        JSON.stringify({
          type: "join",
          id: id,
        })
      );
    };
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      console.log(data);
      if (data.type == "stream") {
        console.log(data);
        streamText += data.content;
        chatScroll();
      }
      if (data.type == "done") {
        typing = false;
        messages = [
          ...messages,
          {
            id: Date.now().toString(),
            role: "assistant",
            content: streamText,
          },
        ];
        chatScroll();
      }
      if (data.type == "join") {
        character = data.state.character;
        char_id = data.state.id;
        ai_role = character.name;
        messages = data.state.messages;
        chatToLocalStorage();
        chatScroll();
      }
    };
    ws.onclose = () => {
      console.log("WebSocket connection closed");
      reconnectWebSocket();
    };
  };
  const reconnectWebSocket = () => {
    setTimeout(() => {
      connectWebSocket();
    }, 1000);
  };
  connectWebSocket();
</script>

<div class="topbar">
  <div class="hamburger" on:click={() => toggleLeftSidebar()}>
    <svg
      data-slot="icon"
      aria-hidden="true"
      fill="none"
      stroke-width="1.5"
      stroke="currentColor"
      viewBox="0 0 24 24"
      xmlns="http://www.w3.org/2000/svg"
    >
      <path
        d="M3.75 6.75h16.5M3.75 12h16.5m-16.5 5.25h16.5"
        stroke-linecap="round"
        stroke-linejoin="round"
      ></path>
    </svg>
  </div>
  <div class="title">VFriend</div>
  <div class="logout" on:click={() => logout()}>
    <svg
      data-slot="icon"
      aria-hidden="true"
      fill="none"
      stroke-width="1.5"
      stroke="currentColor"
      viewBox="0 0 24 24"
      xmlns="http://www.w3.org/2000/svg"
    >
      <path
        d="M15.75 9V5.25A2.25 2.25 0 0 0 13.5 3h-6a2.25 2.25 0 0 0-2.25 2.25v13.5A2.25 2.25 0 0 0 7.5 21h6a2.25 2.25 0 0 0 2.25-2.25V15m3 0 3-3m0 0-3-3m3 3H9"
        stroke-linecap="round"
        stroke-linejoin="round"
      ></path>
    </svg>
  </div>
</div>
<div
  style="display: flex; width: 100vw; height: calc(100vh - 80px); gap: 10px; justify-content: space-between; margin-top: 60px;"
>
  <div class="leftsidebar">
    <IdCard
      id_image={character.id_image}
      name={character.name}
      subtitle={character.subtitle}
    />
    <CharList />
    {#if char_id}
      <div
        style="  border: 1px solid var(--border) ;width: calc(100% - 10px); background-color: var(--background); margin-top: 10px; border-radius: 5px; display: flex; justify-content: center; align-items: center; padding: 5px; gap: 5px;"
      >
        <button on:click={newchat}>New Chat</button>
      </div>
    {/if}
  </div>
  <div class="flex-column">
    <div class="chatscreen">
      <div class="charinfo mobile-only">
        <img src={character.id_image} alt="" />
        <div class="title">{character.name}</div>
        <div class="subtitle">{character.subtitle}</div>
      </div>
      <div id="chats">
        {#each messages as message}
          {#if message.role === "user"}
            <div class="usermessage">
              <strong class="username blue">You</strong>
              {@html converter.makeHtml(message.content)}
            </div>
          {:else if message.role === "assistant"}
            <div class="assistantmessage">
              <strong class="username yellow">{ai_role} </strong>
              {@html converter.makeHtml(message.content)}
              <div class="opts">
                {#if message.content === messages[messages.length - 1].content}
                  <div class="small-btn" on:click={() => regen_last()}>
                    <svg
                      data-slot="icon"
                      aria-hidden="true"
                      fill="none"
                      stroke-width="1.5"
                      stroke="currentColor"
                      viewBox="0 0 24 24"
                      xmlns="http://www.w3.org/2000/svg"
                    >
                      <path
                        d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0 3.181 3.183a8.25 8.25 0 0 0 13.803-3.7M4.031 9.865a8.25 8.25 0 0 1 13.803-3.7l3.181 3.182m0-4.991v4.99"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                      ></path>
                    </svg>
                  </div>
                {/if}
              </div>
            </div>
          {/if}
        {/each}
        {#if typing}
          <strong class="username glowing yellow"
            >{ai_role}
            <div class="spinner">⁐</div></strong
          >
          <div class="assistantmessage">
            {@html converter.makeHtml(streamText)}
          </div>
        {/if}
      </div>
    </div>

    <div class="chatinput">
      <textarea
        placeholder="Type a message..."
        bind:value={textinput}
        on:keydown={(e) => {
          enterKeyHandler(e);
        }}
      ></textarea>
      <button on:click={send}
        ><svg
          data-slot="icon"
          aria-hidden="true"
          fill="currentColor"
          viewBox="0 0 20 20"
          xmlns="http://www.w3.org/2000/svg"
        >
          <path
            d="M3.105 2.288a.75.75 0 0 0-.826.95l1.414 4.926A1.5 1.5 0 0 0 5.135 9.25h6.115a.75.75 0 0 1 0 1.5H5.135a1.5 1.5 0 0 0-1.442 1.086l-1.414 4.926a.75.75 0 0 0 .826.95 28.897 28.897 0 0 0 15.293-7.155.75.75 0 0 0 0-1.114A28.897 28.897 0 0 0 3.105 2.288Z"
          ></path>
        </svg></button
      >
    </div>
  </div>
  <div class="leftsidebar">
    <ChatHistory />
  </div>
</div>

<style>
  .title {
    flex-grow: 1;
    text-align: center;
  }
  .logout {
    width: 50px;
    text-align: center;
    cursor: pointer;
  }
  .logout > svg {
    width: 30px;
    margin-top: 10px;
    stroke: var(--text-muted);
  }
  .charinfo {
    text-align: center;
    gap: 10px;
    margin-bottom: 10px;
    width: 100%;
  }
  .charinfo > img {
    height: 100px;
    width: 100px;
    border-radius: 50px;
  }
  .charinfo .title {
    font-size: 20px;
    font-weight: bold;
  }
  .charinfo .subtitle {
    font-size: 15px;
    color: var(--text-muted);
  }
  .small-btn {
    height: 30px;
    width: 30px;
    width: fit-content;
    font-size: 13px;
    color: #ccc;
    padding: 0;
    cursor: pointer;
  }
  .small-btn > svg {
    height: 20px;
    stroke: var(--text-muted);
  }
  .small-btn:hover > svg {
    stroke: var(--links);
  }
  .hamburger {
    cursor: pointer;
    height: 40px;
    width: 40px;
    margin: 5px;
    display: none;
  }
  .spinner {
    animation: spin 1s linear infinite;
    display: inline-block;
    margin-left: 10px;
  }

  @keyframes spin {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
  .leftsidebar {
    top: 60px;
    width: 400px;
    left: 10px;
    height: 100%;
    margin-left: 5px;
  }
  .username {
    font-weight: bold;
    margin-right: 10px;
  }
  .usermessage {
    border-bottom: #414141 solid 1px;
  }

  .assistantmessage {
    border-top-left-radius: 0px;
    border-bottom: #414141 solid 1px;
  }
  .assistantmessage::before {
    content: "";
    width: 0;
    height: 0;
  }

  .flex-column {
    display: flex;
    flex-direction: column;
    flex-grow: 1;
    align-items: center;
    gap: 10px;
    max-width: 60%;
  }

  .topbar {
    z-index: 10;
    height: 50px;
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    font-size: 20px;
    line-height: 50px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #383838;
    color: #ffffff;
    font-weight: bold;
  }

  .chatscreen {
    width: calc(100% - 40px);
    background-color: var(--background);
    padding: 10px 20px;
    flex-grow: 1;
    overflow-y: scroll;
    overflow-x: hidden;
    border-radius: 5px;
    display: flex;
    flex-direction: column;
  }

  .chatinput {
    width: calc(100%);
    height: 120px;
    color: var(--button-base);
    display: flex;
    background-color: var(--background);
    border-radius: 5px;
    gap: 10px;
    align-items: center;
  }

  .chatinput > textarea {
    flex-grow: 1;
    height: 100px;
    background-color: var(--background);
    border-radius: 5px;
    padding: 10px;
    resize: none;
    outline: none;
  }

  .chatinput > textarea:focus {
    box-shadow: none;
  }

  .chatinput > button {
    height: 60px;
    width: 60px;
    padding: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 40px;
  }
  .chatinput > button > svg {
    height: 26px;
    stroke: var(--text-main);
    fill: var(--text-main);
  }
  .mobile-only {
    display: none;
  }
  @media (max-width: 600px) {
    .mobile-only {
      display: block;
    }
    .hamburger {
      display: block;
    }
    .topbar {
    }
    .leftsidebar {
      display: none;
    }
    .flex-column {
      max-width: calc(100% - 10px);
      margin-left: 5px;
    }
  }
</style>
