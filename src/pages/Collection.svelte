<script>
  let server_url = import.meta.env.VITE_SERVER_URL;
  const newchat = async (id) => {
    const response = await fetch(
      server_url
        ? `https://${server_url}/newchat`
        : "http://localhost:3000/newchat",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          id: id,
        }),
      }
    );
    const data = await response.json();
    if (data.id) {
      window.location.href = `/chat/${data.id}`;
    }
  };
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
<div style="margin: 30px; margin-top: 100px">
  <h1>Characters</h1>
</div>

<div style="margin: 100px; display: flex; flex-wrap: wrap; gap: 20px">
  <div class="char">
    NARUTO
    <button on:click={() => newchat("naruto")}>Start Chat</button>
  </div>
  <div class="char">
    GORDON RAMSAY

    <button on:click={() => newchat("gordon")}>Start Chat</button>
  </div>
  <div class="char">
    GAL GADOT
    <button on:click={() => newchat("gal")}>Start Chat</button>
  </div>
</div>

<style>
  .char {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 300px;
    height: 400px;
    background-color: var(--background);
    border: 1px solid var(--background-alt);
    border-radius: 5px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }
  .char > button {
    margin-top: 100px;
  }
  .hamburger {
    cursor: pointer;
    height: 40px;
    width: 40px;
    margin: 5px;
    display: none;
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

  .mobile-only {
    display: none;
  }
  @media (max-width: 600px) {
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
  .title {
    font-size: 20px;
    font-weight: bold;
    flex-grow: 1;
    text-align: center;
  }
</style>
