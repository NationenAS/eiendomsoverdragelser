<script>

import { onMount } from "svelte"
// @ts-ignore
import { each } from "svelte/internal"
// @ts-ignore
import * as L from "leaflet"
// @ts-ignore
import { MarkerClusterGroup } from 'leaflet.markercluster';
import * as turf from "@turf/turf"
import Modal from "./Modal.svelte"
import Locate from "./Locate.svelte"
import List from "./List.svelte"
import Lock from "./Lock.svelte"
import Help from "./Help.svelte"
import Filter from "./Filter.svelte";

export let initialSale
export let showList

let map,
	sales = [], // Data array with sales objects
	salesList = [],
	salesInView = [],
	activeSalePolygons = L.featureGroup(),
	salesMarkerGroup = new MarkerClusterGroup({ showCoverageOnHover: false, zoomToBoundsOnClick: true }),
	showModal = false,
	locked = false,
	isMobile = L.Browser.mobile,
	saleIcon = L.divIcon({ className: 'map-marker', html: '<svg><use xlink:href="#saleIcon"></svg>' })
$:	activeSale = {}

function getSalesInView() {
	salesInView = []
	let b = map.getBounds()
	salesList.forEach(s => {
    	if (b.contains([ s[4][0], s[4][1] ])) salesInView.push(s)
	})
}

// Map setup
function createMap(container) {
    map = L.map(container)
		.setView([61, 9], 7)
		.on('moveend', update)
	if (isMobile) {
		setLock(true)
		console.log("User on mobile, locked.")
	}
	L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
		maxZoom: 19,
		attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
	}).addTo(map)
}
function populateMap(filters) {
	let filteredSales
	// Reset
	salesList = []
	salesMarkerGroup.clearLayers().remove()
	// Filter
	filters ? filteredSales = filterArr(sales, filters) : filteredSales = sales
	// Create sale markers and list
	for (let sale of filteredSales) {
		if(sale.prop[0].coord != null) { // TODO!!
			let layer = L.marker(sale.prop[0].coord, {
					icon: saleIcon
				})
				.on('click', () => {
					markerClick(sale.saleId)
					console.log("SaleId: " + sale.saleId)
				})
			layer.addTo(salesMarkerGroup)
			salesList.push([
				sale.saleId,
				sale.date.substring(0,10),
				sale.prop[0].municipality,
				sale.price,
				sale.prop[0].coord,
				sale.type
			])
		}
	}
	// Add to map
	salesMarkerGroup.addTo(map)
	salesList = salesList
	// Focus initial sale on map
	if (initialSale != null && !filters) {
		markerClick(initialSale)
	}
	// Update
	update()
}
function resizeMap() { if (map) { map.invalidateSize() } }
function update() {
	getSalesInView()
}

// Get data
onMount(async () => {
	fetch("https://eiendomsoverdragelser-vercel.vercel.app/api/sales")
		.then(response => response.json())
		.then(data => {
			sales = data
			populateMap()
			getSalesInView()			
		})
		.catch(error => { console.log(error) })
})

// Actions
function markerClick(saleId, fromList = false) {
	// Find array index from SaleId, abort if not found
	let i = sales.findIndex(x => x.saleId == saleId)
	if (i == -1) return
	// If from list and on mobile, jump
	if (window.innerWidth < 680 && fromList) {
		window.scrollBy({ top: document.querySelector('#Eiendomsoverdragelser').getBoundingClientRect().top - 50 })
	}
	// Reset
	if (activeSalePolygons.getLayers() != null) activeSalePolygons.clearLayers().remove()
	// @ts-ignore
	activeSale = {}
	// Register properties which share area/teig to prevent area duplicates
	let matNumbTexts = []
	// Make layer group and add to map
	activeSalePolygons.addTo(map).on('layeradd', () => {
		map.flyToBounds(activeSalePolygons.getBounds(), {
			duration: 1
		})
	})
	// Add markers to group
	sales[i].prop.forEach(p => {
		fetch("https://ws.geonorge.no/eiendom/v1/geokoding?matrikkelnummer=" + p.matNumb + "&omrade=true&utkoordsys=4326")
			.then(r => r.json())
			.then(data => {
				// Create marker if has geo and is not duplicate
				if(data.features.length > 0 && !matNumbTexts.includes(data.features[0].properties.matrikkelnummertekst)) {
                    // Takes all features of all matrikkelnumbers, union area, and update area calculation
                    data.features.forEach(e => {
                        // @ts-ignore
                        if (!activeSale.features) activeSale.features = e
                        // @ts-ignore
                        else activeSale.features = turf.union(activeSale.features, e)
                        // @ts-ignore
                        activeSale.area = turf.area(activeSale.features)
                    })
					L.geoJSON(data, { style: { color: "red", fillOpacity: 0.4, weight: 0, } })
						.bindPopup(() => {
							let txt = p.matNumb + "<br>" + Math.round(turf.area(data) / 1000) + " dekar"
							if (p.address) txt = p.address + "<br>" + txt
							return txt
						})
						.addTo(activeSalePolygons)
					matNumbTexts.push(data.features[0].properties.matrikkelnummertekst)
				}
			})
	})
	// @ts-ignore
	activeSale.sale = sales[i]
	showModal = true
    console.log(activeSale)
}
function closeModal() { 
	showModal = !showModal
	let center = map.getCenter()
	activeSalePolygons.clearLayers().remove()
	if (map.getZoom() > 10) map.flyTo(center, 10, { duration: 0.5 })
}
function locateUser() {
	showModal = false
	map.locate({setView: true, maxZoom: 9});
}
function setLock(state) {
	if (state) {
		map._handlers.forEach((h) => { h.disable() })
		locked = true
	} else {
		map._handlers.forEach((h) => { h.enable() })
		locked = false
	}
}
function filterArr(sales, config) { 
    let filteredSales = [];
    sales.forEach(sale => {
      let containing = [];

      //filterer på pris
      if(config.price.filter === true) {
        if(parseInt(sale.price) < parseInt(config.price.min) || parseInt(sale.price) >= parseInt(config.price.max))  return       
      }

      //filtrerer på type salg
      if(config.type.filter == true) {
        if(!config.type.type.includes(sale.type) )  return
      }

      if(config.prop.filter == true) {
        
        sale.prop.forEach( p => {
          if(config.prop.type.includes(p.type) ) containing.push(p.type)
        })

        if (containing.length == 0) return
      }
      //console.log(sale.type, sale.price, containing);
      filteredSales.push(sale)
    });
    return filteredSales
}

</script>

<svelte:window on:resize={resizeMap} />
<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="true">
	<link href="https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;600;700&display=swap" rel="stylesheet">
</svelte:head>

<main>

<Help />
<div class="map-container">
	<div class="map" use:createMap>
		{#if showModal}<Modal {activeSale} on:close={closeModal} />{/if}
		<Filter on:setFilters={(event) => populateMap(event.detail.config) } />
		<Locate on:locate={locateUser} />
		{#if isMobile}<Lock on:toggleLock={(event) => { setLock(event.detail.state) }} {locked} />{/if}
	</div>
</div>
<div class="description">
	Beregnet eiendomsareal kan avvike noe fra faktisk areal. Informasjon/kartdata om eiendommene leveres av Kartverket.
</div>
{#if showList == "show"}
<List rows={salesInView} on:rowClick={(event) => markerClick(event.detail.id, true)} />
{/if}

</main>

<svg xmlns="http://www.w3.org/2000/svg" style="display: none;">
	<symbol id="saleIcon" viewBox="0 0 23 33.6">
		<path d="M11.5,0c6.3-.1,12,6.1,11.5,12.5-.1,3.2-2,5.8-3.5,8.5-2.1,3.7-6.7,11.8-7,12.1-.5,.8-1.7,.7-2.1,0-.5-.9-5.8-10-7.9-13.7C1.3,17.3,.1,15,0,12.6-.5,6.2,5.1,0,11.5,0c0,0,0,0,0,0ZM6.4,12c0,2.9,2.3,5,4.8,5.1,7.2,0,7.2-10.2,.2-10.3-2.6,0-5,2.3-5,5.1Z"/>
	</symbol>
</svg>

<style>
:global(main){
    --green: #406619;
	--dark-green: #3a4e20;
}
main {
	font-family: "Open Sans", sans-serif;
	font-size: 16px;
	width: 980px;
	max-width: 100%;
	margin: 0 auto;
}
.map-container {
	position: relative;
}
.map {
	height: 560px;
	border-radius: 3px;
}
.description {
	margin: 5px auto;
	font-size: 0.8em;
	color: #666;
	line-height: 1.25;
}
</style>