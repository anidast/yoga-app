<script>
  import { Router, Route } from "svelte-routing";
  import Hero from "./lib/Hero.svelte";
  
	export let url= "";
</script>

<body>
  <Router {url}>
    <Route path="/" component={Hero} />
  </Router>
</body>

<style>
  body{
    background-color: #EDEFFE;
    font-family: 'Poppins', sans-serif;
  }
</style>