<script>

import { createEventDispatcher } from 'svelte'
import Locked from "./Icons/Locked.svelte"
import Unlocked from "./Icons/Unlocked.svelte"

export let locked

const dispatch = createEventDispatcher()
function toggle(state) { dispatch( 'toggleLock', { state: state } ) }

</script>

{#if locked}
<div class=locked>
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div style="cursor:pointer" on:click={() => { toggle(false) }}>
        Klikk for å navigere i kartet.
    </div>
</div>
{:else}
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class=unlocked on:click={() => { toggle(true) }}>
    <Unlocked />
</div>
{/if}

<style>
.unlocked {
    position: absolute;
    top: 0;
    right: 0;
    padding: 20px;
    z-index: 10001;
    cursor: pointer;
}
.locked {
    position: absolute;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    z-index: 10001;
    background: rgba(255, 255, 255, 0.45);
    font-size: 15px;
    font-family: adelle_sansbold;
}
.locked div {
    background: var(--green);
    font-family: "Open Sans", sans-serif;
    font-weight: 600;
    padding: 0.67rem 1rem;
    color: white;
    border: none;
    border-radius: 3px;
    box-shadow: 1px 1px 8px rgba(0, 0, 0, 0.35);
    cursor: pointer;
}
.locked div:hover {
    background: var(--dark-green);
}
</style>