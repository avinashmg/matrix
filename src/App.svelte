<script>
  import { onMount } from "svelte";
  import Chat from "./pages/Chat.svelte";
  import { Client, Account } from "appwrite";
  import { navigate, Route, Router } from "svelte-routing";
  import Collection from "./pages/Collection.svelte";
  import Login from "./pages/Login.svelte";
  const client = new Client();
  client.setProject("v-friend");
</script>

<Router>
  <Route path="/" component={Collection} />
  <Route path="/chat/:id" let:params>
    <Chat id={params.id} />
  </Route>
  <Route path="/login" component={Login} />
</Router>
