<script lang="ts">
	import VindProd from "./component/VindProd.svelte";
	import type {Production} from "./interfaces/interface";
	export let consumption:number = 0;
	let production:number = 0;
	
	let ppp: Production;
	let pro: Production[] =[];
	
	let pa:number[] =[];
	let pa1:number[][] =[];
	let consumArray: number[][]=[];
	let h0 = 0;
	// let h1 = 1;
	let d0 = 1;
	// let d1 = 1;
	let t=-1;//neg værdi bruges til initiallisering
	let i =0;
	// let prod =0;
	var date0 = new Date(2021, 1, 1, 0, 0, 0);
	// var date1 = new Date(2021, 1, 2, 1, 0, 0);
	
	function pad(d) {
    	return (d < 10) ? '0' + d.toString() : d.toString();	
	}
	$:if (t<0) {//lidt klodset måde at fortage den initielle indlæsning af data
		t=date0.getDate();
		fetchdata(t);
	}
	function add() {
		//optaterer tiden
		date0.setHours(date0.getHours()+1)
		h0=date0.getHours(); //ny time
		d0=date0.getDate();	//evt ny dato
		// date1.setHours(date1.getHours()+1)
		// h1=date1.getHours();
		// d1=date1.getDate();
		
		if(t != d0) {//t er global dato - 
			t= d0;
			fetchdata(t);
		}else{
			production=pro[h0].vind+pro[h0].sol+pro[h0].orc;
			ppp=pro[h0];
			console.log(date0,production," pa1:",pro[h0]);
			consumption=-1;
		}

	} 
	function fetchdata(d:number) {
	
		fetch('https://api.energidataservice.dk/dataset/ProductionMunicipalityHour?offset=0&start=2021-01-'+pad(d)+'T00:00&end=2021-01-'+pad(d+1)+'T00:00&filter=%7B%22MunicipalityNo%22:[%22492%22]%7D&sort=HourUTC%20ASC&timezone=dk')
		.then(response => response.json())
		.then(data=> {
			// let sum = 0;
			data.records.forEach((record,n) => {
				// const d = [];
				const p:Production ={vind:0,sol:0,orc:0,tid:""};
				// sum += record.OnshoreWindMWh;
				// sum += record.SolarMWh;
				// sum += record.ThermalPowerMWh;
				// d[0]=(record.OnshoreWindMWh)*1000;
				// d[1]=(record.SolarMWh)*1000;
				// d[2]=(record.ThermalPowerMWh)*1000;
				p.vind=(record.OnshoreWindMWh)*1000;
				p.orc=(record.ThermalPowerMWh)*1000;
				p.sol=(record.SolarMWh)*1000;
				p.tid = record.HourDK;
				pro[n]=p;
				// pa[n]=sum*1000;
				console.log(pro[n],n,record);
				if(n==0){
					production = p.vind+p.sol+p.orc;
					ppp = p;
				}
				// sum=0
        });
		
		console.log("******");
    });

	// 	fetch('https://api.energidataservice.dk/dataset/PrivIndustryConsumptionHour?offset=0&start=2021-01-'+pad(d0)+'T'+pad(h0)+':00&end=2021-01-'+pad(d1)+'T'+pad(h1)+':00&filter=%7B%22MunicipalityNo%22:[%22492%22]%7D&timezone=dk')
    // .then(response => response.json())
    // .then(data=> {
    //     let sum = 0;
	// 	let j=0;
    //     data.records.forEach(record => {
    //         sum += record.ConsumptionkWh;
    //         //  console.log( record.ConsumptionkWh,sum,j++);
    //     });
	// 	consumption = sum;
	// 	// console.log("******");
    // });
}

</script>

<main>
	<h1>{pad(d0)}/{date0.getMonth()}-{date0.getFullYear()} {pad(h0)}:00</h1>
	<button on:click={add}>+++</button>
	<p>Visit the <a href="https://svelte.dev/tutorial">Svelte tutorial</a> to learn how to build Svelte apps.</p>
	<VindProd sum = {Math.round(consumption)} prod = {Math.round(production)} production={ppp}></VindProd>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}
	

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>