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

    let people = $state([]);
    let N = $state(1);

    const groups = $state([]);
    let selectedGroupIdx = $state(0);
    let subtotals = $derived(
        groups.map((g) => g.items.reduce((acc, i) => acc + i.cost, 0)),
    );
    let selectedCnt = $derived(
        groups.map((g) => g.checked.reduce((acc, c) => acc + (c ? 1 : 0), 0)),
    );
    let personalTotal = $derived.by(() => {
        let res = [];
        console.log(subtotals);
        console.log(selectedCnt);
        for (let i = 0; i < N; i++) {
            let total = 0;
            for (let gi = 0; gi < groups.length; gi++) {
                if (!groups[gi].checked[i]) continue;

                total += subtotals[gi] / selectedCnt[gi];
            }
            res.push(total);
        }
        return res;
    });

    $inspect(personalTotal);
    $inspect(groups);

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

    function NChange(e) {
        N = e.target.value;

        if (N - 1 > people.length) {
            let pushed = false;
            for (const debtor of config.debtors) {
                if (!people.includes(debtor.name)) {
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

            {#each groups as group, groupIdx}
                <tr>
                    <td
                        onclick={() => {
                            selectedGroupIdx = groupIdx;
                        }}
                        colspan="2"
                        class={groupIdx == selectedGroupIdx ? "selected" : ""}
                    >
                        {group.name}
                    </td>
                    {#each { length: N }, i}
                        <td>
                            <input
                                type="checkbox"
                                bind:checked={groups[groupIdx].checked[i]}
                            />
                        </td>
                    {/each}
                </tr>
                {#each group.items as item, itemIdx}
                    <tr>
                        <td
                            onclick={() => {
                                selectedGroupIdx = groupIdx;
                            }}
                            class={groupIdx == selectedGroupIdx
                                ? "selected"
                                : ""}
                        >
                            {group.name}
                        </td>
                        <td> {item.name} </td>
                        <td colspan={N}>
                            <input type="number" bind:value={item.cost} />
                        </td>
                    </tr>
                {/each}
                <tr>
                    <td
                        onclick={() => {
                            selectedGroupIdx = groupIdx;
                        }}
                        colspan="2"
                        class={groupIdx == selectedGroupIdx ? "selected" : ""}
                    >
                        그룹 소계
                    </td>
                    <td colspan={N}> {subtotals[groupIdx]} </td>
                </tr>
            {/each}

            <tr>
                <td>
                    <button onclick={addGroup}> 그룹+ </button>
                </td>
                <td>
                    <button onclick={addItem}> 항목+ </button>
                </td>
                {#each { length: N }, i}
                    <td> {personalTotal[i]} </td>
                {/each}
            </tr>
        </tbody>
    </table>
    <button> 후잉 업로드 </button>
</main>
