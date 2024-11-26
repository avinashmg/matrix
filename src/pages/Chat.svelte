<script>
  import showdown from "showdown";
  import IdCard from "../lib/IdCard.svelte";
  let textinput = "";
  let messages = [];
  let ai_role = "Donald Trump";
  const initial_messages = [
    {
      role: "system",
      content:
        "You are a Donald Trump. You are now elected as President 2024. Roleplay as Donald Trump convincingly, in the style he speaks.",
    },
    {
      role: "assistant",
      content:
        "I am Mr President Elect of Unites States of America, I have great words, the best words",
    },
  ];
  let typing = false;
  let streamText = "";

  const converter = new showdown.Converter();

  function parseChunk(chunk) {
    const lines = chunk.split("\n").filter((line) => line.trim() !== "");
    let parsedText = "";
    for (const line of lines) {
      if (line.startsWith("data: ")) {
        const json = line.slice(6);
        if (json !== "[DONE]") {
          const content = JSON.parse(json).choices[0].delta?.content;
          if (content) parsedText += content;
        }
      }
    }
    return parsedText;
  }
  const send = async () => {
    if (typing) return;
    if (textinput === "") return;
    messages = [
      ...messages,
      {
        id: Date.now().toString(),
        role: "user",
        content: textinput,
      },
    ];

    textinput = "";
    typing = true;
    const response = await fetch("http://127.0.0.1:8080/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        stream: true,
        temperature: 0.8,
        messages: [...initial_messages, ...messages],
        cache_prompt: true,
        samplers: "dkypmxt",
        dynatemp_range: 0,
        dynatemp_exponent: 1,
        top_k: 40,
        top_p: 0.95,
        min_p: 0.05,
        typical_p: 1,
        xtc_probability: 0,
        xtc_threshold: 0.1,
        repeat_last_n: 64,
        repeat_penalty: 1,
        presence_penalty: 0,
        frequency_penalty: 0,
        dry_multiplier: 0,
        dry_base: 1.75,
        dry_allowed_length: 2,
        dry_penalty_last_n: -1,
        max_tokens: -1,
      }),
    });

    const reader = response.body.getReader();
    const decoder = new TextDecoder("utf-8");
    streamText = "";
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      const chunk = decoder.decode(value);
      const parsedContent = parseChunk(chunk);
      streamText += parsedContent;
    }
    messages = [
      ...messages,
      {
        id: Date.now().toString(),
        role: "assistant",
        content: streamText,
      },
    ];
    typing = false;
    streamText = "";
    document.getElementsByClassName("chatscreen")[0].scrollTo(0, 9999);
  };

  const enterKeyHandler = (e) => {
    if (!e.shiftKey && e.key === "Enter") {
      e.preventDefault();
      send();
    }
  };
</script>

<div class="topbar">AI CHAT</div>
<div class="leftsidebar">
  <IdCard />
</div>
<div class="flex-column">
  <div class="chatscreen">
    {#each messages as message}
      {#if message.role === "user"}
        <div class="usermessage">
          <strong class="username">You</strong>
          {@html converter.makeHtml(message.content)}
        </div>
      {:else if message.role === "assistant"}
        <div class="assistantmessage">
          <strong class="username">{ai_role}</strong>
          {@html converter.makeHtml(message.content)}
        </div>
      {/if}
    {/each}
    {#if typing}
      <strong class="username">{ai_role}</strong>
      <div class="assistantmessage">{@html converter.makeHtml(streamText)}</div>
    {/if}
  </div>

  <div class="chatinput">
    <textarea
      bind:value={textinput}
      on:keydown={(e) => {
        enterKeyHandler(e);
      }}
    ></textarea>
    <button on:click={send}>Send</button>
  </div>
</div>

<style>
  .leftsidebar {
    position: fixed;
    top: 60px;
    width: 400px;
    left: 10px;
    height: calc(100vh - 70px);
  }
  .username {
    font-weight: bold;
    margin-right: 10px;
    color: var(--text-muted);
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
    position: fixed;
    top: 0;
    display: flex;
    flex-direction: column;
    height: 100vh;
    align-items: center;
    gap: 10px;
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
    text-align: center;
    background-color: #383838;
    color: #ffffff;
    font-weight: bold;
  }

  .chatscreen {
    width: 600px;
    margin-top: 60px;
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
    width: calc(600px + 40px);
    height: 120px;
    color: var(--button-base);
    display: flex;
    gap: 10px;
  }

  .chatinput textarea {
    flex-grow: 1;
    height: 100px;
    background-color: var(--background);
    border-radius: 5px;
    padding: 10px;
    resize: none;
  }

  .chatinput button {
    height: 100px;
  }
</style>
