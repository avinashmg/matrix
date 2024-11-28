<script>
  import { onMount } from "svelte";
  import Chat from "./pages/Chat.svelte";
  import { Client, Account } from "appwrite";
  import { Route, Router } from "svelte-routing";
  import Collection from "./pages/Collection.svelte";
  import Login from "./pages/Login.svelte";
  const client = new Client();
  client.setProject("v-friend");

  const checkLogin = async () => {
    if (window.location.pathname === "/login") return;
    const account = new Account(client);
    console.log(account);
    try {
      const user = await account.get();
      loggedIn = true;
    } catch (error) {
      window.location.href = "/login";
    }
  };
  checkLogin();
</script>

<Router>
  <Route path="/" component={Collection} />
  <Route path="/chat/:id" let:params>
    <Chat id={params.id} />
  </Route>
  <Route path="/login" component={Login} />
</Router>
