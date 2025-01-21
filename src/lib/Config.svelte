<script>
    import { config } from "../config.svelte";

    if (!config.configured) {
        config.show = true;
    }

    function showConfig() {
        config.show = true;
    }

    function addDebtor() {
        config.debtors.push({ name: "", debit: "" });
    }

    function deleteDebtor(idx) {
        config.debtors.splice(idx, 1);
    }

    function addExpense() {
        config.expenses.push("");
    }
    function deleteExpense(idx) {
        config.expenses.splice(idx, 1);
    }

    function addAsset() {
        config.assets.push("");
    }
    function deleteAsset(idx) {
        config.assets.splice(idx, 1);
    }

    function save() {
        localStorage.setItem("endpoint", config.endpoint);
        localStorage.setItem("bond", config.bond);
        localStorage.setItem("debtors", JSON.stringify(config.debtors));
        localStorage.setItem("expenses", JSON.stringify(config.expenses));
        localStorage.setItem("assets", JSON.stringify(config.assets));

        config.show = false;
        config.configured = true;
    }

    let modalClass = $derived(
        "modal".concat(config.show ? " modal-active" : ""),
    );
</script>

<div>
    <button onclick={showConfig}>설정</button>
    <div id="config-modal" class={modalClass}>
        <div id="config-body" class="modal-body">
            <label for="endpoint">endpoint</label>
            <input
                type="text"
                id="endpoint"
                defaultValue={config.endpoint}
                bind:value={config.endpoint}
            />
            <br />
            <label for="bond">
                <abbr
                    title="후잉에서 빌려준 돈을 저장하는 자산 항목 이름(거래처 관리 항목이어야 함)"
                >
                    빌려준돈
                </abbr>
            </label>
            <input
                type="text"
                id="bond"
                defaultValue={config.bond}
                bind:value={config.bond}
            />
            <br />
            <table border="1">
                <tbody>
                    <tr>
                        <th>비용</th>
                    </tr>
                    {#each config.expenses as expense, idx}
                        <tr>
                            <td>
                                <input
                                    type="text"
                                    defaultValue={expense}
                                    bind:value={config.expenses[idx]}
                                />
                            </td>
                            <td>
                                <button onclick={() => deleteExpense(idx)}>
                                    X
                                </button>
                            </td>
                        </tr>
                    {/each}
                    <tr>
                        <td> <button onclick={addExpense}> + </button> </td>
                    </tr>
                </tbody>
            </table>
            <table border="1">
                <tbody>
                    <tr>
                        <th>자산</th>
                    </tr>
                    {#each config.assets as asset, idx}
                        <tr>
                            <td>
                                <input
                                    type="text"
                                    defaultValue={asset}
                                    bind:value={config.assets[idx]}
                                />
                            </td>
                            <td>
                                <button onclick={() => deleteAsset(idx)}>
                                    X
                                </button>
                            </td>
                        </tr>
                    {/each}
                    <tr>
                        <td> <button onclick={addAsset}> + </button> </td>
                    </tr>
                </tbody>
            </table>
            <table border="1">
                <tbody>
                    <tr>
                        <th>이름</th>
                        <th>차변</th>
                        <th></th>
                    </tr>
                    {#each config.debtors as row, idx}
                        <tr>
                            <td>
                                <input
                                    type="text"
                                    defaultValue={row.name}
                                    bind:value={row.name}
                                />
                            </td>
                            <td>
                                <input
                                    type="text"
                                    defaultValue={row.debit}
                                    bind:value={row.debit}
                                />
                            </td>
                            <td>
                                <button
                                    onclick={() => {
                                        deleteDebtor(idx);
                                    }}>X</button
                                >
                            </td>
                        </tr>
                    {/each}
                    <tr>
                        <td colspan="2">
                            <button onclick={addDebtor}>추가</button>
                        </td>
                    </tr>
                </tbody>
            </table>
            <button onclick={save}>저장</button>
        </div>
    </div>
</div>

<style>
    #config-body {
        border: 2px solid black;
        padding: 1em;
        margin: 1em;
    }
</style>
