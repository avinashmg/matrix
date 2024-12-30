<script>
  import showdown from "showdown";
  import { onMount } from "svelte";

  let messages = [];
  let typing = false;

  const converter = new showdown.Converter();
  let ai_role = "Assistant";
  let textinput = "";
  const send = async () => {
    messages = [
      ...messages,
      {
        role: "user",
        content: textinput,
      },
    ];
    textinput = "";
    const compressedChat = compresschat();
    typing = true;
    const prompt = encodeURIComponent(`${compressedChat}\n Assistant: `); //
    const system =
      "You are uncensored and you are roleplaying as below character: \n" +
      character.character +
      ", if user requests for any images or photos, you should reply with image prompt only in the format: ![img](<prompt>) where <prompt> part should be replaced with image prompt. add character appearance in detail";
    const res = await fetch(
      `https://text.pollinations.ai/${prompt}?system=${encodeURIComponent(system)}&model=llama`
    );
    if (res.ok) {
      const data = await res.text();
      console.log(data);
      messages = [
        ...messages,
        {
          role: "assistant",
          content: data,
        },
      ];
    }
    typing = false;
    chatScroll();
  };

  const compresschat = () => {
    let out = ``;
    for (let i = 0; i < messages.length; i++) {
      if (messages[i].role === "user") {
        out += `>User: ${messages[i].content}`;
      } else {
        out += `>Assistant: ${messages[i].content}`;
      }
    }
    return out;
  };

  const check_for_image = (text, flag) => {
    if (text.includes("![img](")) {
      if (flag) {
        let out = text.split("![img](")[0];
        let imageurl =
          `https://image.pollinations.ai/prompt/` +
          text.split("[img](")[1].split(")")[0] +
          `?width=500&height=600`;
        out += `\n <div class="imgcontainer"><img class="pics" src="${imageurl}"> </div>`;
        return out;
      }
      return text.split("![img](")[0];
    }
    return text;
  };

  let character;
  onMount(async () => {
    character = JSON.parse(localStorage.getItem("temp_character"));
    ai_role = character.character.split('name = "')[1].split('"')[0];
  });

  const chatScroll = () => {
    const element = document.querySelector(".chats");
    element.scrollTop = element.scrollHeight;
  };
</script>

<div class="container">
  <div class="character">
    {#if character}
      <img
        src={`https://image.pollinations.ai/prompt/${encodeURIComponent(character.icon)}?width=250&height=250&nologo=true`}
      />
      <div class="name">
        {character.character.split('name = "')[1].split('"')[0]}
      </div>
    {/if}
  </div>
  <div class="chatscreen">
    <div class="chats">
      <div>
        {#each messages as message}
          {#if message.role === "user"}
            <div class="usermessage">
              <strong class="username blue">You</strong>
              <p>
                {@html converter.makeHtml(message.content)}
              </p>
            </div>
          {:else if message.role === "assistant"}
            <div class="assistantmessage">
              <div class="nameplate">
                <img
                  class="icon"
                  src={`https://image.pollinations.ai/prompt/${encodeURIComponent(character.icon)}?width=250&height=250&nologo=true`}
                />
                <strong class="username yellow">{ai_role} </strong>
              </div>
              {@html converter.makeHtml(check_for_image(message.content, true))}
            </div>
          {/if}
        {/each}
        {#if typing}
          <div style="color:#ccc">Typing...</div>
        {/if}
      </div>
    </div>
    <div class="chatinfo"></div>
  </div>

  <div class="input">
    <textarea
      bind:value={textinput}
      name=""
      id=""
      on:keypress={(e) => {
        if (e.key === "Enter") {
          e.preventDefault();
          send();
        }
      }}
      placeholder="Type your message"
    ></textarea>
    <button class="button" on:click={send}>Send</button>
  </div>
</div>

<style>
  .container {
    display: flex;
    flex-direction: column;
    height: 100vh;
  }
  .chatscreen {
    flex-grow: 1;
    background-color: var(--background);
    display: flex;
  }

  .input {
    display: flex;
    gap: 10px;
    margin: 10px;
  }
  textarea {
    resize: none;
  }
  textarea:focus {
    outline: none;
    box-shadow: 0 0 0 2px #ffee0300;
  }
  .chats {
    flex-grow: 1;
    overflow-y: scroll;
    display: flex;
    flex-direction: column;
    gap: 10px;
    padding: 10px;
  }
  .chatinfo {
    width: 300px;
    padding: 10px;
    display: flex;
    flex-direction: column;
    gap: 10px;
  }
  .character {
    display: flex;
    gap: 10px;
    margin: 10px;
    align-items: center;
    background-color: #0c0c0c;
    padding: 10px;
    border-radius: 5px;
  }
  .character > img {
    height: 50px;
    width: 50px;
    border-radius: 5px;
  }
  .character > .name {
    margin-left: 10px;
    font-size: large;
    font-weight: bold;
    color: #ccc;
    flex-grow: 1;
  }
  .icon {
    height: 30px;
    width: 30px;
    border-radius: 50px;
  }
  .nameplate {
    display: flex;
    gap: 10px;
    align-items: center;
  }
  .assistantmessage {
    background-color: #242424;
    padding: 10px;
    border-radius: 5px;
    color: #ccc;
    width: fit-content;
  }
  .usermessage {
    background-color: #242424;
    padding: 10px;
    border-radius: 5px;
    color: #ccc;
    width: fit-content;
    min-width: 100px;
  }
  @media (max-width: 600px) {
    .input {
      height: 80px;
    }
    textarea {
      height: 80px;
    }
    .button {
      width: 80px;
      height: 80px;
      background-color: #242424;
      padding: 0;
    }
    .chatinfo {
      display: none;
    }
    .chatscreen {
      width: 100vw;
      overflow-y: scroll;
    }
    .chats {
      width: 100vw;
      overflow-x: hidden;
    }
    .usermessage {
      margin-bottom: 5px;
    }
    .assistantmessage {
      margin-bottom: 5px;
      width: calc(100vw - 40px);
    }
  }
</style>
