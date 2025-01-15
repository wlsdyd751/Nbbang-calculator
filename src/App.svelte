<script>
    import Dropdown from "./lib/Dropdown.svelte";
    import Config from "./lib/Config.svelte";
    import { config } from "./config.svelte";

    if (localStorage.getItem("endpoint") !== null) {
        config.endpoint = localStorage.getItem("endpoint");
        config.bond = localStorage.getItem("bond");
        config.debtors = JSON.parse(localStorage.getItem("debtors"));
        config.expenses = JSON.parse(localStorage.getItem("expenses"));
        config.assets = JSON.parse(localStorage.getItem("credits"));
        config.configured = true;
    }

    let people = $state([]);
    let N = $state(1);

    let selectedCredit = $state("");
    let selectedExpense = $state("");
    let item = $state("");
    let today = new Date();
    let dateString = $state("");
    let date = $derived(dateString.replace(/-/g, ""));
    let memo = $state("");

    const groups = $state([]);
    let selectedGroupIdx = $state(0);
    let subtotals = $derived(
        groups.map((g) => g.items.reduce((acc, i) => acc + i.cost, 0)),
    );
    let selectedCnt = $derived(
        groups.map((g) => g.checked.reduce((acc, c) => acc + (c ? 1 : 0), 0)),
    );
    let { personalTotal, actualTotal } = $derived.by(() => {
        let p = [];
        let a = [];
        for (let i = 0; i < N; i++) {
            p.push(0);
            a.push(0);
        }
        for (let [gi, group] of groups.entries()) {
            for (let i = 0; i < N; i++) {
                if (!group.checked[i]) continue;
                a[i] += subtotals[gi] / selectedCnt[gi];
                p[i] += Math.floor(subtotals[gi] / selectedCnt[gi]);
            }

            let indexes = [];
            for (let i = 0; i < N; i++) {
                indexes.push(i);
            }
            console.log("idxx");
            console.log(indexes);
            indexes.sort((i, j) => {
                return p[i] - a[i] - p[j] + a[j];
            });
            console.log("idxsorted");
            console.log(indexes);

            for (let i = 0; i < subtotals[gi] % selectedCnt[gi]; i++) {
                p[indexes[i]] += 1;
            }
        }
        return { personalTotal: p, actualTotal: a };
    });

    $inspect(personalTotal);

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

    function submitToWhooing() {
        for (let i = 0; i < N; i++) {
            if (personalTotal[i] > 0) {
                let left;
                let right = selectedCredit;
                let dataItem = item;
                if (i == 0) {
                    left = selectedExpense;
                } else {
                    let name = people[i - 1];
                    let debit = config.debtors.find(
                        (d) => d.name === name,
                    ).debit;
                    console.log(`debit: //${debit}//`);

                    if (debit === "") {
                        left = config.bond;
                        dataItem = name;
                    } else {
                        left = debit;
                    }
                }
                const data = {
                    entry_date: parseInt(date),
                    item: dataItem,
                    money: personalTotal[i],
                    left,
                    right,
                    memo,
                };
                console.log("data");
                console.log(JSON.stringify(data));

                fetch(config.endpoint, {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify(data),
                });
            }
        }
    }
</script>

<main>
    <Config />
    <label for="N"> 인원수 </label>
    <input type="number" id="N" onchange={NChange} defaultValue="1" />
    <br />
    왼쪽
    <Dropdown bind:value={selectedExpense} options={config.expenses} />
    오른쪽
    <Dropdown bind:value={selectedCredit} options={config.assets} />
    <br />
    <label for="item"> 아이템 </label>
    <input type="text" id="item" placeholder="아이템" bind:value={item} />
    <br />
    <label for="date"> 날짜 </label>
    <input type="date" id="date" bind:value={dateString} />
    <br />
    <label for="memo"> 메모 </label>
    <input type="text" id="memo" placeholder="메모" bind:value={memo} />
    <table border="1">
        <tbody>
            <tr>
                <th colspan="2"> </th>
                <th> 나 </th>
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
            <tr>
                <td colspan="2"> diff </td>
                {#each { length: N }, i}
                    <td> {(personalTotal[i] - actualTotal[i]).toFixed(2)} </td>
                {/each}
            </tr>
        </tbody>
    </table>
    <button onclick={submitToWhooing}> 후잉 업로드 </button>
</main>
