<script>
    let { name, options } = $props();
</script>

<select {name}>
    {#each options as o}
        <option value={o}>{o}</option>
    {/each}
</select>
