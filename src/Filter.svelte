<script>

import { createEventDispatcher } from 'svelte'
const dispatch = createEventDispatcher()

let priceMin,
    priceMax,
    saleType = "Alle",
    propType = "Alle"

function setConfig() {

    let salesConfig = {
        price: {
			filter: false,
			min: priceMin,
			max: priceMax
		},
		type: {
			filter: false,
			type: [ "Fritt salg" ]
		},
		prop: {
			filter: false,
			type: []
		}
    }

    // Filter price
    let min = priceMin
    let max = priceMax

    if ((min.value != "") || (max.value != "") ) {
        salesConfig.price.filter = true
        if(min.value === '') {salesConfig.price.min = 0} else {salesConfig.price.min = parseInt(min.value)}
        if(max.value === '') {salesConfig.price.max = 1000000000} else {salesConfig.price.max = parseInt(max.value)}      
    } else {
        salesConfig.price.filter = false
    }

    // Filter fritt salg
    salesConfig.type.filter = saleType !== "Alle" ? true : false

    // Filter prop
    if (propType !== "Alle") {
      salesConfig.prop.filter = true
      switch(propType) {
        case "Bolig":
            salesConfig.prop.type = ['Frittliggende bolig', 'Rekkehus/kjede', 'Tomannsbolig' ]
            break
        case "Annen":
            salesConfig.prop.type = ['Annen bygning.']
            break
        case "Ubebygget":
            salesConfig.prop.type = ['Ubebygget']
            break
      } 
    } 
    else {
      salesConfig.prop.filter = false
    }  

    dispatch( 'setFilters', { config: salesConfig } )
    hidden = true
}

function reset() {
    priceMin = ''
    priceMax = ''
    saleType = "Alle"
    propType = "Alle"
    setConfig()
}

let hidden = false

</script>

<button class="filter-button" class:hidden={!hidden} on:click={() => { hidden = false }} on:keypress={() => { hidden = false }}>
    Filtrer salgene
</button>

<div class="filter-modal" class:hidden={hidden}>
    <div class="modal-nav">
        <svg on:click={() => { hidden = true }} on:keypress={(event) => { hidden = true }} viewBox="0 0 10 10"><polyline points="1,9 9,1 5,5 1,1 9,9"></polyline></svg>
    </div>
    <form>
        <div class="input-group">
            <div class="group-heading">Salgspris</div>
            <label for="price-min"><span class="fromto">Fra</span><input type="number" id="price-min" name="price-min" min="0" max="10000000" step="100000" bind:this={priceMin}> kr</label>
            <label for="price-max"><span class="fromto">Til</span><input type="number" id="price-max" name="price-max" min="0" max="100000000" step="100000" bind:this={priceMax}> kr</label>
        </div>
        <div class="input-group">
            <div class="group-heading">Omsetningstype</div>
            <label for="alle-salg"><input type="radio" id="alle-salg" name="sales-type" bind:group={saleType} value={"Alle"}> Alle</label>   
            <label for="fritt-salg"><input type="radio" id="fritt-salg" name="sales-type" bind:group={saleType} value={"Fritt salg"}> Fritt salg</label>   
        </div>
        <div class="input-group">
            <div class="group-heading">Bebyggelse</div>
            <label for="alle"><input type="radio" id="alle" name="prop-type" bind:group={propType} value={"Alle"}> Alle</label>
            <label for="bolig"><input type="radio" id="bolig" name="prop-type" bind:group={propType} value={"Bolig"}> Bolig</label>
            <label for="annen"><input type="radio" id="annen" name="prop-type" bind:group={propType} value={"Annen"}> Annen bebyggelse</label>
            <label for="ubebygd"><input type="radio" id="ubebygd" name="prop-type" bind:group={propType} value={"Ubebygget"}> Ubebygd</label>
        </div>
        <div class="submit">
            <button type="submit" on:click={(event) => {event.preventDefault(); setConfig()}} on:keypress={(event) => {event.preventDefault(); setConfig()}}>Filtrer</button>
            <div class="reset" on:click={reset} on:keypress={reset}>Nullstill</div>
        </div>
    </form>
</div>

<style>
.filter-button, 
.filter-modal {
    font-family: "Open Sans", sans-serif;
}
.filter-button {
    position: absolute;
    left: 10px;
    bottom: 10px;
    background: white;
    z-index: 9999;
    border-radius: 4px;
    border: 2px solid rgba(0,0,0,0.35);
    cursor: pointer;
    padding: 5px 8px;
    font-size: 14px;
    font-weight: 600;
}
.filter-button:hover {
    background: #f4f4f4;
}
.filter-modal {
    font-size: 14px;
    line-height: 1.35;
    position: absolute;
    width: 250px;
    max-width: 40%;
    bottom: 30px;
    left: 30px;
    padding: 25px 30px;
    z-index: 10000;
    background: white;
    color: black;
    border-radius: 5px;
    box-shadow: 1px 1px 8px rgba(0, 0, 0, 0.35);
    transition: .4s ease-in;
    accent-color: var(--green);
    --vertical: 15px;
}
form {
    display: flex;
    flex-direction: column;
    gap: var(--vertical);
}
.hidden {
    display: none;
}
.modal-nav {
    position: absolute;
    cursor: pointer;
    padding: 10px;
    right: 20px;
    top: 15px;
    z-index: 10002;
}
svg {
    width: 20px;
}
polyline {
    stroke: #666;
    stroke-width: 1.25px;
    stroke-linecap: round;
    stroke-linejoin: round;
    fill: none;
}
.group-heading {
    margin-bottom: 5px;
    font-size: 15px;
    font-weight: bold;
}
.fromto {
    width: 30px;
    display: inline-block;
}
label {
    display: block;
    cursor: pointer;
    font-weight: normal;
    margin: 0;
}
input[type=number] {
    margin-bottom: 2px;
    border: none;
    border-bottom: 1px solid black;
    padding: 2px;
    width: 120px;
    text-align: right;
    outline: none;
    font-size: 15px;
    -moz-appearance: textfield;
    appearance: textfield;
}
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
}
input[type=radio] {
    margin-right: 3px;
}
.submit {
    display: flex;
    gap: 10px;
    align-items: center;
}
.submit button {
    background: var(--green);
    color: white;
    border: none;
    border-radius: 3px;
    padding: 0.5rem;
    display: block;
    cursor: pointer;
    flex: 1;
}
button:hover {
    background-color: var(--dark-green);
}
.reset {
    text-decoration: underline;
    font-size: 13px;
    cursor: pointer;
}
.reset:hover {
    color: darkgrey;
}
@media (max-width: 630px) {
    .filter-modal {
        left: 10px;
        bottom: 10px;
        right:10px;
        max-width: unset;
        width: unset;
        --vertical: 10px;
    }
}
</style>