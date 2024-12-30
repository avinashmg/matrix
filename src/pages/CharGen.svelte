<script>
  import showdown from "showdown";

  let textinput = "";
  let character = "";
  let image = "";
  let image2 = "";
  const send = async () => {
    character = "";
    image = "";
    image2 = "";
    const prompt = encodeURIComponent(
      textinput +
        "\n\n A character based on above description in toml format:\n\n "
    );
    const res = await fetch(`https://text.pollinations.ai/${prompt}`);
    const data = await res.text();
    character = "```" + data.split("```")[1] + "```";
    console.log(data);
    const image_prompt = encodeURIComponent(
      data.split("```toml")[1].split("```")[0] +
        "\n Create an image generation prompt for above character's profile picture. \n\n Image generation prompt: "
    );
    const image_res = await fetch(
      `https://text.pollinations.ai/${image_prompt}`
    );
    image = await image_res.text();

    const image_prompt2 = encodeURIComponent(
      data.split("```toml")[1].split("```")[0] +
        "\n Create an image generation prompt for above character's profile picture with only face. \n\n Image generation prompt: "
    );
    const image_res2 = await fetch(
      `https://text.pollinations.ai/${image_prompt2}`
    );

    image2 = await image_res2.text();
  };

  const converter = new showdown.Converter();

  const save = async () => {
    const saved_characters = JSON.parse(
      localStorage.getItem("saved_characters") ?? "[]"
    );
    if (saved_characters.find((c) => c.character == character)) {
      alert("Character already saved");
      return;
    }
    saved_characters.push({ character: character, pic: image, icon: image2 });
    localStorage.setItem("saved_characters", JSON.stringify(saved_characters));
    alert("Character saved");
  };
</script>

<div class="container">
  <h1>Character Generator</h1>
  <textarea placeholder="Enter the prompt" bind:value={textinput} name="" id=""
  ></textarea>
  <button on:click={send}>Generate</button>

  {#if character}
    <div>
      <h2>Character</h2>
      {#if image}
        <img
          class="image"
          src={"https://image.pollinations.ai/prompt/" +
            encodeURIComponent(image) +
            "?width=250&height=250&nologo=true"}
          alt=""
        />
      {/if}
      {#if image2}
        <img
          class="image"
          src={"https://image.pollinations.ai/prompt/" +
            encodeURIComponent(image2) +
            "?width=250&height=250&nologo=true"}
          alt=""
        />
      {/if}
      <br />
      <button on:click={() => save()}>Save Character</button>
      {@html converter.makeHtml(character)}
    </div>
  {/if}
</div>

<style>
  .container {
    margin: 10px;
  }
  textarea {
    resize: none;
  }
  .image {
    height: 250px;
    width: 250px;
    background-image: url("/loading.gif");
    background-size: cover;
    background-position: center;
    border-radius: 10px;
    margin: 10px;
    border: 1px solid var(--border);
  }
</style>
