<script>
    import Dropdown from "./lib/Dropdown.svelte";
    import Config from "./lib/Config.svelte";
    import { config } from "./config.svelte";

    if (localStorage.getItem("endpoint") !== null) {
        config.endpoint = localStorage.getItem("endpoint");
        config.bond = localStorage.getItem("bond");
        config.debtors = JSON.parse(localStorage.getItem("debtors"));
        config.expenses = JSON.parse(localStorage.getItem("expenses"));
        config.configured = true;
    }
</script>

<main>
    <Config />
</main>
