<script>
  import { Account, Client, ID } from "appwrite";
  import { onMount } from "svelte";
  let iti = null;
  let loggedIn = undefined;

  const client = new Client();
  client.setProject("v-friend");
  onMount(async () => {
    const input = document.querySelector("#phone");
    window.intlTelInput(input, {
      loadUtilsOnInit:
        "https://cdn.jsdelivr.net/npm/intl-tel-input@24.7.0/build/js/utils.js",
    });
    iti = window.intlTelInput(input, {
      initialCountry: "auto",
      geoIpLookup: (callback) => {
        fetch("https://ipapi.co/json")
          .then((res) => res.json())
          .then((data) => callback(data.country_code))
          .catch(() => callback("us"));
      },
      utilsScript: "/intl-tel-input/js/utils.js?1730730622316", // just for formatting/placeholders etc
    });
  });

  let phone = "";
  let userId;
  let SecretCode;
  const signin = async () => {
    const account = new Account(client);
    const token = await account.createPhoneToken(
      ID.unique(),
      `+${iti.s.dialCode}${phone}`
    );

    userId = token.userId;
    console.log(token);
  };

  const phoneVerify = async () => {
    const account = new Account(client);
    try {
      const session = await account.createSession(userId, SecretCode);
      console.log(session);
      localStorage.setItem("session", JSON.stringify(session));
      loggedIn = true;
    } catch (error) {
      console.log(error);
      alert("Wrong Code, Please enter the correct code");
    }
  };
</script>

<div class="signin">
  {#if !userId}
    <h1>Enter using Phone</h1>
    {#if iti}
      {iti.getNumber()}
    {/if}
    <div class="phone">
      <input
        bind:value={phone}
        id="phone"
        type="tel"
        placeholder="Phone number"
      />
    </div>
    <button on:click={signin}>Sign in</button>
  {:else}
    <h1>Enter the Code</h1>
    <p>A code has been sent to {phone}. Please enter it below</p>
    <input bind:value={SecretCode} type="text" placeholder="Code" />
    <button on:click={phoneVerify}>Submit </button>
  {/if}
</div>

<style>
  .signin {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
  }

  .phone {
    margin: 20px;
  }
</style>
