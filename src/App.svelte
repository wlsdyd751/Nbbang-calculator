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
    addGroup();

    function addItem() {
        groups[selectedGroupIdx].items.push({
            name: groups[selectedGroupIdx].items.length + 1,
            cost: 0,
        });
    }

    let people = $state([]);
    let N = $state(1);

    function NChange(e) {
        N = e.target.value;
        console.log(N);

        if (N - 1 > people.length) {
            console.log(`people: ${people}`);
            let pushed = false;
            for (const debtor of config.debtors) {
                console.log(`debtor: ${debtor.name}`);
                if (!people.includes(debtor.name)) {
                    console.log("pushed");
                    people.push(debtor.name);
                    pushed = true;
                    break;
                }
            }

            if (!pushed) {
                N -= 1;
            }
        }

        if (N < 1) {
            N = 1;
        }

        if (N - 1 < people.length) {
            people.pop();
        }
        e.target.value = N;
    }
</script>

<main>
    <Config />
    <label for="N">인원수</label>
    <input type="number" id="N" onchange={NChange} defaultValue="1" />
    <table border="1">
        <tbody>
            <tr>
                <th colspan="2"> </th>
                <th> me </th>
                {#each { length: N - 1 }, i}
                    <th>
                        <Dropdown
                            bind:value={people[i]}
                            options={config.debtors.map((d) => d.name)}
                        />
                    </th>
                {/each}
            </tr>

            {#each groups as group, idx}
                <tr>
                    <td
                        onclick={() => {
                            selectedGroupIdx = idx;
                        }}
                        colspan="2"
                        class={idx == selectedGroupIdx ? "selected" : ""}
                    >
                        {group.name}
                    </td>
                    {#each { length: N }, i}
                        <td>
                            <input type="checkbox" />
                        </td>
                    {/each}
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
