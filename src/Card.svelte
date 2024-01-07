<script>
    import { createEventDispatcher } from 'svelte';
    export let data;
    export let cardSpace;
    const dispatch = createEventDispatcher();

    function mouseEnterCard() {
        dispatch('message', {
            type: 'mouseenter',
            data: data,
        });
    }
    function mouseLeaveCard() {
        dispatch('message', {
            type: 'mouseleave',
            data: data
        });
    }
    function clickCard() {
        dispatch('message', {
            type: 'click',
            data: data,
        });
    }

</script>

<div class="cardspacer" style="height: {cardSpace}px; margin-top: -{cardSpace/4}px">
    <div class="card" style="transform: rotate({(Math.random()*1.5) -1}deg) translateX({Math.random()*10}px);" >
        <img src={data[data.active]} alt="" on:mouseenter={mouseEnterCard} on:mouseleave={mouseLeaveCard} on:click={clickCard}>
    </div>
</div>
