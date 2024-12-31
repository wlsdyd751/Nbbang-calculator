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

    const groups = $state([]);
    let selectedGroupIdx = $state(0);

    function addGroup() {
        groups.push({
            name: `${groups.length + 1}`,
            checked: [],
            items: [],
        });
        selectedGroupIdx = groups.length - 1;
    }

    function addItem() {
        groups[selectedGroupIdx].items.push({
            name: groups[selectedGroupIdx].items.length + 1,
            cost: 0,
        });
    }
</script>

<main>
    <Config />
    <table border="1">
        <tbody>
            <tr>
                <th colspan="2"> </th>
                <th> </th>
            </tr>

            {#each groups as group, idx}
                <tr>
                    <td colspan="2"> {group.name} </td>
                    <td> </td>
                    <td
                        onclick={() => {
                            selectedGroupIdx = idx;
                        }}
                        colspan="2"
                        class={idx == selectedGroupIdx ? "selected" : ""}
                    >
                        {group.name}
                    </td>
                </tr>
                {#each group.items as item, idx}
                    <tr>
                        <td colspan="2"> {group.name}-{item.name} </td>
                        <td colspan={N}>
                            <input type="number" bind:value={item.cost} />
                        </td>
                    </tr>
                {/each}
            {/each}

            <tr>
                <td>
                    <button onclick={addGroup}> 그룹+ </button>
                </td>
                <td>
                    <button onclick={addItem}> 항목+ </button>
                </td>
            </tr>
        </tbody>
    </table>
</main>
