<script>
	// Components for working with Mapbox layers
	import { getData, getColor, getTopo } from "./js/utils.js";
	import Map from './Map.svelte';
	import MapSource from './MapSource.svelte';
	import MapLayer from './MapLayer.svelte';
	import MapTooltip from './MapTooltip.svelte';
	import ColorBar from './ColorBar.svelte';
	
	const colors = {
		seq5: ['rgb(234, 236, 177)', 'rgb(169, 216, 145)', 'rgb(0, 167, 186)', 'rgb(0, 78, 166)', 'rgb(0, 13, 84)'],
		div10: ['#67001f','#b2182b','#d6604d','#f4a582','#fddbc7','#d1e5f0','#92c5de','#4393c3','#2166ac','#053061']	
	};

	const pconData = "./data/salary-pcon10.csv";
	const pconBounds = {
	  url: "./data/pcon10-bounds.json",
	  layer: "PCONreg",
	  code: "AREACD"
	};
	
	const lsoaData = "./data/imd-lsoa11.csv";
	const lsoaBounds = {
		url: "https://cdn.ons.gov.uk/maptiles/administrative/lsoa/v1/boundaries/{z}/{x}/{y}.pbf",
		layer: "boundaries",
		code: "AREACD"
	};

	const bounds = {
		uk: [[ -9, 49 ], [ 2, 61 ]],
		ew: [[-6, 49], [2, 56]]
	};

	const baseMaps = [
		{
			key: "omt",
			label: "OpenMapTiles",
			path: "./data/style-ons-light.json"
		},
		{
			key: "osm",
			label: "OpenStreetMap",
			path: "./data/style-osm-grey.json"
		},
	];
	
	// Bindings
	let map1, map2, map3, map4;

	// Data
	let data = {};
	let geojson;

	// State
	let zoom;
	let center = {};
	let hovered, selected;

	let showSources = true;
	let showLayers = true;
	let visLayers = true;
	let baseMap = baseMaps[0];

	let breaks;

	// Get geometry for geojson maps
	getTopo(pconBounds.url, pconBounds.layer)
	.then(res => geojson = res);
	
	// Get data for geojson maps
	getData(pconData)
	.then(res => {
		let vals = res.map(d => d.salary).sort((a, b) => a - b);
		let len = vals.length;
		breaks = [
			vals[0],
			vals[Math.floor(len * 0.2)],
			vals[Math.floor(len * 0.4)],
			vals[Math.floor(len * 0.6)],
			vals[Math.floor(len * 0.8)],
			vals[len - 1]
		];
		res.forEach(d => {
			d.color = getColor(d.salary, breaks, colors.seq5);
		});
		data.pcon = res;
	});

	// Get data for vector tiles map
	getData(lsoaData)
	.then(res => {
		res.forEach(d => {
			d.color = colors.div10[+d.income_decile - 1];
			d.AREACD = d.lsoa11cd;
		});
		data.lsoa = res;
	});

	let hoveredValue = null;
	let hoveredAREANM = null;
	let selectedValue = null;

	function handleHover(event) {
		if (event.detail && event.detail.feature) {
			const hoveredAREACD = event.detail.feature.properties.AREACD;
			getData(pconData)
			.then(res => {
				const feature = res.find(d => d.AREACD === hoveredAREACD);
				hoveredValue = feature ? feature.salary : null;
				hoveredAREANM = feature ? feature.AREANM : null;
			});	
		} else {
			hoveredValue = null;
		}
	}

	function handleSelect(event) {
		if (event.detail && event.detail.feature) {
			const selectedAREACD = event.detail.feature.properties.AREACD;
			getData(pconData)
			.then(res => {
				const feature = res.find(d => d.AREACD === selectedAREACD);
				selectedValue = feature ? feature.salary : null;
			});
		} else {
			selectedValue = null;
		}
	}
</script>


<div>
	<div class="map">
	{#if geojson && data.pcon}
	<Map id="map3" style={baseMap.path} location={{bounds: bounds.uk}} bind:map={map3} controls={true}>
			{#if showSources}
		<MapSource
			id="pcon"
			type="geojson"
			data={geojson}
			promoteId={pconBounds.code}
			maxzoom={13}>
				{#if showLayers}
			<MapLayer
				id="pcon-fill"
				data={data.pcon}
				type="fill"
				hover={true}
				bind:hovered
				select={true}
				bind:selected
				on:hover={handleHover}
				on:select={handleSelect}
				paint={{
					'fill-color': ['case',
						['!=', ['feature-state', 'color'], null], ['feature-state', 'color'],
						'rgba(255, 255, 255, 0)'
					],
					'fill-opacity': 0.7
				}}
					order={baseMap.key === "omt" ? "water_name" : null}
					visible={visLayers}
			>
				<MapTooltip content={`<b>${hoveredAREANM}</b>: £${hoveredValue}`}/>
				</MapLayer>
			<MapLayer
				id="pcon-line"
				type="line"
				paint={{
					'line-color': ['case',
						['==', ['feature-state', 'selected'], true], 'black',
						['==', ['feature-state', 'hovered'], true], 'orange',
						'rgba(255, 255, 255, 0)'
					],
					'line-width': ['case',
						['==', ['feature-state', 'selected'], true], 2,
						1
					]
				}}
					visible={visLayers}
				/>
				{/if}
		</MapSource>
			{/if}
	</Map>
	<ColorBar colors={colors.seq5} breaks={breaks} title="Salary (£)" hoveredValue={hoveredValue} selectedValue={selectedValue} />
	{/if}
	</div>
	<div class="map-info">
		Geojson choropleth map with hover and select<br/>
		(hovered: {hovered ? hovered : ''},
		selected: {#if selected} {selected} <button on:click|preventDefault={() => selected = null}>x</button>{/if})
	</div>
</div>

<style>
	button {
		padding: 0 2px;
		cursor: pointer;
	}
	.map {
		height: 100vh;
		/* border: 1px solid #ccc; */
		position: relative;
	}
	.map-info {
		position: absolute;
		bottom: 10px;
		left: 10px;
		background-color: #d6d6d6cc;
		padding: 10px;
		border-radius: 4px;
		font-size: 0.9em;
		z-index: 1000;
	}
</style>
