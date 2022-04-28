<script>

	let database;
	function getJSON(url) {
  		var xmlhttprequest = new XMLHttpRequest();
		xmlhttprequest.open('GET', url, true);
		xmlhttprequest.responseType = 'json';
		xmlhttprequest.onload = function() {
			database = xmlhttprequest.response;
			fillTimeframe(database);
			fillTable(database, 0);
		};
		xmlhttprequest.send();
	}

	getJSON('https://api.carbonintensity.org.uk/regional/intensity/source/2022-02-14T18:00:00.000Z/fw24h');

	function fillTimeframe(db) {
		let selector = document.getElementById('time_selector');
		selector.innerHTML = '';
		for(let i = 0; i < db.data.length; i++) {
			let option = document.createElement("option");
			option.text = `${db.data[i].from} to ${db.data[i].to}`;
			option.value = i;
			selector.add(option);
		}
	}

	function setTimeframe() {
		let selector = document.getElementById('time_selector');
		fillTable(database, selector.value);
		closeCard();
	}

	let intensityIndexColor = ['#67cfb0', '#afe0ad', '#ffd561', '#ff7657', '#c44d6f'];
	let currentData = [];
	let currentLabels = [];
	let currentData2 = [];
	let currentLabels2 = [];

	function fillTable(db, t) {
		//console.log(db);
		let tbody = document.getElementById('tbody');
		tbody.innerHTML = '';
		let legend1 = document.getElementById('chart_legend');
		let legend2 = document.getElementById('chart2_legend');

		for(let i = 0; i < db.data[t].regions.length; i++) {
			let row = document.createElement('tr');
			row.setAttribute('id', `row${i}`);
			let col1 = document.createElement('td');
			col1.setAttribute('id', `col${i}_1`);
			let col2 = document.createElement('td');
			col2.setAttribute('id', `col${i}_2`);
			let col3 = document.createElement('td');
			col3.setAttribute('id', `col${i}_3`);

			col1.innerHTML = db.data[t].regions[i].dnoregion;
			col2.innerHTML = db.data[t].regions[i].shortname;
			col3.innerHTML = db.data[t].regions[i].intensity.index;

			row.appendChild(col1);
			row.appendChild(col2);
			row.appendChild(col3);
			tbody.appendChild(row);

			col3.style.color = getColor(db.data[t].regions[i].intensity.index);
			row.addEventListener('mouseenter', ()=> {
				row.style.backgroundColor = getColor(db.data[t].regions[i].intensity.index);
				col3.style.color = '#000000';
				col3.style.fontWeight = '400';
			});
			row.addEventListener('mouseleave', ()=> {
				row.style.backgroundColor = '#ffffff';
				col3.style.color = getColor(db.data[t].regions[i].intensity.index);
				col3.style.fontWeight = '600';
			});
			row.addEventListener('click', ()=> {
				regionName = db.data[t].regions[i].shortname;
				carbonIntensity = db.data[t].regions[i].regionid;
				document.getElementById('carbon_intensity').style.color = getColor(db.data[t].regions[i].intensity.index);
				document.getElementById('card').style.transform = 'scale(1)';
				legend1.innerHTML = '';

				for(var j = 0; j < db.data[t].regions[i].generationmix.length; j++) {
					currentLabels[j] = `${db.data[t].regions[i].generationmix[j].fuel}: ${db.data[t].regions[i].generationmix[j].perc}%`;
					currentData[j] = db.data[t].regions[i].generationmix[j].perc;

					if(db.data[t].regions[i].generationmix[j].perc > 0) {
						var row = legend1.insertRow(legend1.rows.length);
						var cell1 = row.insertCell(0);
						var cell2 = row.insertCell(1);
						cell1.innerHTML = '&#11044;';
						cell1.style.color = data.datasets[0].backgroundColor[j];
						cell2.innerHTML = `${db.data[t].regions[i].generationmix[j].fuel}: ${db.data[t].regions[i].generationmix[j].perc}%`;
					}
				}

				data.labels = currentLabels;
				data.datasets[0].data = currentData;
				genmixChart.update();

				legend2.innerHTML = '';

				for(var j = 0; j < db.data[t].regions[i].sourceregion.length; j++) {
					currentLabels2[j] = `${db.data[t].regions[i].sourceregion[j].source}: ${db.data[t].regions[i].sourceregion[j].perc}%`;
					currentData2[j] = db.data[t].regions[i].sourceregion[j].perc;
				}

				let counter = 0;
				for(var j = 0; j < db.data[t].regions[i].sourceregion.length; j++) {
					currentLabels2[j] = `${db.data[t].regions[i].sourceregion[j].source}: ${db.data[t].regions[i].sourceregion[j].perc}%`;
					currentData2[j] = db.data[t].regions[i].sourceregion[j].perc;

					if(db.data[t].regions[i].sourceregion[j].perc > 0 && counter < 9) {
						counter++;
						var row = legend2.insertRow(legend2.rows.length);
						var cell1 = row.insertCell(0);
						var cell2 = row.insertCell(1);
						cell1.innerHTML = '&#11044;';
						cell1.style.color = data2.datasets[0].backgroundColor[j];
						cell2.innerHTML = `${db.data[t].regions[i].sourceregion[j].source}: ${db.data[t].regions[i].sourceregion[j].perc}%`;
					}
				}

				data2.labels = currentLabels2;
				data2.datasets[0].data = currentData2;
				sourceregChart.update();

			});
		}
		
		restyle();
	}

	function restyle() {
		const style = document.createElement('style');
		style.innerHTML = `
			#table {
				box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
				border-radius: 40px;
			}
			td {
				line-height: 40px;
				text-align: center;
				cursor: pointer;
			}
			tr:last-child td:first-child {
				border-radius: 0 0 0 40px;
			}
			tr:last-child td:last-child {
				border-radius: 0 0 40px 0;
			}
			td:last-child {
				font-weight: 600;
			}
			#chart_legend td {
				line-height: 1.5vw;
				text-align: left;
				cursor: default;
			}
			#chart_legend td:first-child {
				width: 1.5vw;
				font-size: 0.8vw;
			}
			#chart_legend td:last-child {
				font-weight: 200;
			}

			#chart2_legend td {
				line-height: 1.5vw;
				text-align: left;
				cursor: default;
			}
			#chart2_legend td:first-child {
				width: 1.5vw;
				font-size: 0.8vw;
			}
			#chart2_legend td:last-child {
				font-weight: 200;
			}
		`;
		document.head.appendChild(style);
	}

	function getColor(intensity) {
		let color;
		if(intensity == 'very low') {
			color = intensityIndexColor[0];
		}
		if(intensity == 'low') {
			color = intensityIndexColor[1];
		}
		if(intensity == 'moderate') {
			color = intensityIndexColor[2];
		}
		if(intensity == 'high') {
			color = intensityIndexColor[3];
		}
		if(intensity == 'very high') {
			color = intensityIndexColor[4];
		}
		return color;
	}

	function closeCard() {
		document.getElementById('card').style.transform = 'scale(0)';
	}

	let regionName = 'England';
	let carbonIntensity = 208;



    import Chart from 'chart.js/auto/auto.js';

    let genmixChartCon;
    let data = {
		labels: [],
        datasets: [{
			data: [],
            backgroundColor: ['#00b294', '#af415b', '#616364', '#ea234d', '#ffdc82', '#c7cfd8', '#3f5377', '#9ad69a', '#6593aa'],
            borderWidth: 0
        }]
    };
    
	const config = {
        type: 'doughnut',
        data: data,
        options: {
            responsive: true,
            cutout: '60%',
            spacing: 0,
            plugins: {
                legend: {
                    position: 'left',
                    display: false,
                    labels: {
                        usePointStyle: true,
                        padding: 10,
                        font: {
                            size: 14
                        }
                    }
                },
				tooltip: {
					callbacks: {
            			label: function(context) {
                        	return context.label;
                    	}
        			}
    			}
            }
        }
    };

	let sourceregChartCon;
    let data2 = {
		labels: [],
        datasets: [{
			data: [],
            backgroundColor: ['#00b294', '#267366', '#af415b', '#820a26', '#616364', '#21343d', '#ea234d', '#f55d7e', '#ffdc82', '#ffc226', '#c7cfd8', '#bbd5f2', '#3f5377', '#1c499c', '#9ad69a', '#6b8f6b', '#6593aa', '#245773'],
            borderWidth: 0
        }]
    };

	const config2 = {
        type: 'doughnut',
        data: data2,
        options: {
            responsive: true,
            cutout: '60%',
            spacing: 0,
            plugins: {
                legend: {
                    position: 'right',
                    display: false,
                    labels: {
                        usePointStyle: true,
                        padding: 10,
                        font: {
                            size: 14
                        }
                    }
                },
				tooltip: {
					callbacks: {
            			label: function(context) {
                        	return context.label;
                    	}
        			}
    			}
            }
        }
    };

	var genmixChart;
	var sourceregChart;
	import { onMount } from 'svelte';

	onMount(async () => {
		const ctx = genmixChartCon.getContext('2d');
        genmixChart = new Chart(ctx, config);

		const ctx2 = sourceregChartCon.getContext('2d');
        sourceregChart = new Chart(ctx2, config2);
	});

</script>








<main>
	<h1>Regional Carbon Intensity</h1>
	<select id="time_selector" on:change={setTimeframe}>
		<option value="timeframe">timeframe</option>
	</select>
	<table id="table">
		<thead>
			<tr>
				<th>DNO Region</th>
				<th>Shortname</th>
				<th>Intensity</th>
			</tr>
		</thead>
		<tbody id="tbody"></tbody>
	</table>
	<div id="card">
		<div id="region_name">{regionName}</div>
		<div id="cci">current carbon intensity</div>
		<div id="carbon_intensity">{carbonIntensity}</div>
		<div id="unit">gCO<sup>2</sup>/kWh</div>
		<div id="close" on:click={closeCard}></div>
		<div id="chart_container">
			<canvas id="chart_canvas" bind:this={genmixChartCon}/>
		</div>
		<table id="chart_legend"></table>
		<div id="chart2_container">
			<canvas id="chart2_canvas" bind:this={sourceregChartCon}/>
		</div>
		<table id="chart2_legend"></table>
	</div>
	
</main>








<style>

	main {
		padding-bottom: 50px;
	}

	h1 {
		color: #666666;
		font-size: 4em;
		font-weight: 100;
		text-align: center;
	}

	#time_selector {
		margin-left: 50vw;
		transform: translateX(-50%);
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
		border-radius: 100px;
		border: 2px solid #444444;
		color: #444444;
		margin-bottom: 20px;
		font-size: 1vw;
		padding: 10px 15px;
	}

	#table {
		margin-left: 20vw;
		width: 60vw;
		border-collapse: collapse;
		margin-bottom: 100px;
	}

	th {
		background-color: #444444;
		color: #ffffff;
		line-height: 40px;
	}

	th:first-child {
  		border-radius: 40px 0 0 0;
		width: 50%;
	}

	th:last-child {
  		border-radius: 0 40px 0 0;
		  width: 22%;
	}

	#card {
		position: fixed;
		top: 100px;
		left: 15vw;
		width: 70vw;
		height: 35vw;
		background-color: #ffffff;
		box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
		border-radius: 40px;
		font-weight: 100;
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
		transform: scale(0);
		transition: transform .5s;
	}

	#region_name {
		position: absolute;
		top: 2vw;
		width: 100%;
		text-align: center;
		color: #444444;
		font-size: 3.5vw;
	}

	#cci {
		position: absolute;
		top: 7vw;
		width: 100%;
		text-align: center;
		color: #666666;
		font-size: 1.4vw;
		font-weight: 200;
	}

	#carbon_intensity {
		position: absolute;
		top: 9vw;
		width: 100%;
		text-align: center;
		color: #ffd561;
		font-size: 3.5vw;
		font-weight: 600;
	}

	#unit {
		position: absolute;
		top: 13vw;
		width: 100%;
		text-align: center;
		color: #aaaaaa;
		font-size: 1.4vw;
		font-weight: 400;
	}

	#close {
		position: absolute;
		right: 2vw;
		top: 2vw;
		width: 2vw;
		height: 2vw;
		opacity: 0.3;
		cursor: pointer;
	}
	#close:hover {
  		opacity: 1;
	}
	#close:before, #close:after {
		position: absolute;
		left: 1vw;
		content: ' ';
		height: 2vw;
		width: 0.1vw;
		background-color: #444444;
	}
	#close:before {
		transform: rotate(45deg);
	}
	#close:after {
		transform: rotate(-45deg);
	}

	#chart_container {
		position: absolute;
		top: 19vw;
		right: 55%;
		width: 13vw;
	}

	#chart_canvas {
		position: absolute;
		right: 0;
	}

	#chart_legend {
		position: absolute;
		top: 25.5vw;
		left: 4vw;
		transform: translateY(-50%);
	}

	#chart2_container {
		position: absolute;
		top: 19vw;
		left: 55%;
		width: 13vw;
	}

	#chart2_canvas {
		position: absolute;
		left: 0;
	}

	#chart2_legend {
		position: absolute;
		top: 25.5vw;
		left: calc(55% + 14vw);
		transform: translateY(-50%);
	}

</style>