<script lang="ts">
	import VindProd from "./component/VindProd.svelte";
	import type { Production, Consumption } from "./interfaces/interface";

	let ppp: Production ={vind: 0, sol: 0, orc: 0,tid: ""};
	let pro: Production[] = [];
	let consum: Consumption = {ConsumptionBolig:0,ConsumptionErhverv:0,HourDK:""};
	let con: Consumption[] = [];

	let h0 = 0;
	let d0 = 1;
	let t = -1; //neg værdi bruges til initiallisering
	var date0 = new Date(2023, 1, 1, 0, 0, 0);

	function pad(d) {
		return d < 10 ? "0" + d.toString() : d.toString();
	}
	$: if (t < 0) {
		//hmmm lidt klodset måde at fortage den initielle indlæsning af data - bør flyttes til en lifetime ting
		t = date0.getDate();
		fetchdata(t);
	}

	function add() {
		//optaterer tiden
		date0.setHours(date0.getHours() + 1);
		h0 = date0.getHours(); //ny time
		d0 = date0.getDate(); //evt ny dato

		if (t != d0) {
			//t er global dato -
			t = d0;
			fetchdata(t);
		} else {
			ppp = pro[h0];
			consum = con[h0];
		}
	}
	function fetchdata(d: number) {
		fetch(
			"https://api.energidataservice.dk/dataset/ProductionMunicipalityHour?offset=0&start=2023-01-" +
				pad(d) +
				"T00:00&end=2023-01-" +
				pad(d + 1) +
				"T00:00&filter=%7B%22MunicipalityNo%22:[%22492%22]%7D&sort=HourUTC%20ASC&timezone=dk"
		)
			.then((response) => response.json())
			.then((data) => {
				data.records.forEach((record, n) => {
					const p: Production = { vind: 0, sol: 0, orc: 0, tid: "" };

					p.vind = record.OnshoreWindMWh * 1000;
					p.orc = record.ThermalPowerMWh * 1000;
					p.sol = record.SolarMWh * 1000;
					p.tid = record.HourDK;

					pro[n] = p;
					// console.log(pro[n],n,record);
					if (n == 0) {
						ppp = p; //sender til kortet som prop
					}
				});

				fetch(
					"https://api.energidataservice.dk/dataset/PrivIndustryConsumptionHour?offset=0&start=2023-01-" +
						pad(d) +
						"T00:00&end=2023-01-" +
						pad(d + 1) +
						"T00:00&filter=%7B%22MunicipalityNo%22:[%22492%22]%7D&sort=HourUTC%20ASC&timezone=dk"
				)
					.then((response) => response.json())
					.then((data) => {
						let sum = 0;
						let j = 0;

						let sumBolig = 0;
						let erhverv = 0;
						let dato = "";
						data.records.forEach((record, n: number) => {
							if (n == 0 && dato == "") {
								dato = record.HourDK; //initialisering
							}
							if (record.HourDK != dato) {
								//Vi har ramt en ny dato - og gemmer de agregerede værdier
								const c: Consumption = {
									ConsumptionBolig: sumBolig,
									ConsumptionErhverv: erhverv,
									HourDK: dato,
								};
								con[j] = {
									ConsumptionBolig: sumBolig,
									ConsumptionErhverv: erhverv,
									HourDK: dato,
								};

								if (j == 0) {
									consum = c;
								}

								//så initialiseres variable og j tælles op
								sumBolig = 0;
								erhverv = 0;
								dato = record.HourDK;
								j++;
							}

							if (record.HeatingCategory == "Erhverv") {
								erhverv += record.ConsumptionkWh;
							} else {
								sumBolig += record.ConsumptionkWh;
							}
						});
						const c: Consumption = {
							ConsumptionBolig: sumBolig,
							ConsumptionErhverv: erhverv,
							HourDK: dato,
						};
						con[j] = c;
					});
			});
	}
</script>

<main>
	<h1 class="oneline">
		{pad(d0)}/{date0.getMonth()}-{date0.getFullYear()}
		{pad(h0)}:00
	</h1>
	<button class="oneline" on:click={add}> Næste time </button>
	<!-- <p>:::::</p> -->
	<VindProd prod={ppp} {consum} />
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		/* max-width: 240px; */
		margin: 0 auto;
	}
	.oneline {
		display: inline;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: 1000px;
		}
	}
</style>
