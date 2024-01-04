<script>
    import { createEventDispatcher } from 'svelte';
    export let cardname;
    const dispatch = createEventDispatcher();

    function handleClick() {
        dispatch('message', {
            cardname: cardname
        });
    }

</script>

<div class="searchitem" on:click={handleClick}>{cardname}</div>
