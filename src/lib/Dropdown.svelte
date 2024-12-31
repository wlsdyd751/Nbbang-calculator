<script>
    let { value = $bindable(), options } = $props();
</script>

<select bind:value>
    {#each options as o}
        <option value={o}>{o}</option>
    {/each}
</select>
