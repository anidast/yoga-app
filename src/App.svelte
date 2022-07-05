<script>
  import { Router, Route } from "svelte-routing";
  import Hero from "./lib/Hero.svelte";
  import Preparation from "./lib/Preparation.svelte";
  
	export let url= "";
</script>

<body>
  <Router {url}>
    <Route path="/" component={Hero} />
    <Route path="/persiapan" component={Preparation} />
  </Router>
</body>

<style>
  body{
    background-color: #EDEFFE;
    font-family: 'Poppins', sans-serif;
  }
</style>