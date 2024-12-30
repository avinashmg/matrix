<script>
  import { navigate } from "svelte-routing";
  import toml from "toml";
  let saved_characters = JSON.parse(
    localStorage.getItem("saved_characters") ?? "[]"
  );
  for (let i = 0; i < saved_characters.length; i++) {
    const element = saved_characters[i];
    console.log(element);
    console.log(element.character.split('name = "')[1].split('"')[0]);
  }
  const chat_with_char = (character) => {
    localStorage.setItem("temp_character", character);
    navigate("/temp_chat");
  };
</script>

<div>
  {#each saved_characters as character}
    <div class="characters">
      <img
        src={`https://image.pollinations.ai/prompt/${encodeURIComponent(character.pic)}?width=250&height=250&nologo=true`}
      />
      <img
        src={`https://image.pollinations.ai/prompt/${encodeURIComponent(character.icon)}?width=250&height=250&nologo=true`}
      />
      <div class="name">
        {character.character.split('name = "')[1].split('"')[0]}
      </div>
      <button on:click={() => chat_with_char(JSON.stringify(character))}
        >Chat</button
      >
      <button>Add To Main</button>
    </div>
  {/each}
</div>

<style>
  .characters {
    display: flex;
    gap: 10px;
    margin: 10px;
    align-items: center;
    background-color: #0c0c0c;
    padding: 10px;
    border-radius: 5px;
  }
  .characters > img {
    height: 100px;
    width: 100px;
    border-radius: 5px;
  }
  .characters > button {
    background-color: #2b2b2b;
  }
  .name {
    margin-left: 10px;
    font-size: large;
    font-weight: bold;
    color: #ccc;
    flex-grow: 1;
  }
</style>
