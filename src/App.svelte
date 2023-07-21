<script lang="ts">
	import imgUrl from './circuitos.jpg'
	import { onMount } from 'svelte';	
	let datos : {Temperatura_Celcius: number, Humedad: number, Fecha: string}[];
	let dispositivos : string[]
	let dispositivoSeleccionado : string	

	async function fetchData(YaSeleccionado : string | null) {
      //fetch dispositivos
      const res = await fetch(`${import.meta.env.VITE_SERVERHOST}/dispositivos`);
      dispositivos = await res.json();  
      //se inicializa el select al primer dispositivo 
      dispositivoSeleccionado = YaSeleccionado ?? dispositivos[0]
      await fetchRegistros()
    }

    async function fetchRegistros() {
      const res = await fetch(`${import.meta.env.VITE_SERVERHOST}/registros/${dispositivoSeleccionado}`);
      datos = await res.json();
      console.log("datos recibidos: ", datos.length)
    }

	onMount(() => {
		document.body.style.backgroundImage = `url(${imgUrl})`;
		fetchData(null).then(() => {
			//const interval = setInterval(fetchRegistros, 2000);
			//return () => clearInterval(interval);
		})
	});

	let columns
  	$:{
		if(datos){

			if(datos[0].Humedad != 0){
				columns = ["Temperatura ºC", "Humedad %", "Fecha"]
			}
			else{
				columns = ["Temperatura ºC", "Fecha"]
			}
		}
	}
</script>

<svelte:head>
	<title>TP Baldrich y Ramirez</title>
	<meta name="robots" content="noindex nofollow" />
	<html lang="en" />
</svelte:head>

<main>
  <h1>Sistemas de Hardware para la Administracion</h1>
  <h3>Trabajo Práctico - Fernando Baldrich y Thiago Ramirez</h3>

  {#key datos}
  <div class="card">
    {#if !datos}
	<h2 id="tempInfo">Cargando...</h2>
    {:else if datos.length === 0}
	{console.log("datos: ", datos)}
      <h2 id="tempInfo">No hay datos aún</h2>
    {:else}

    <h2>Dispositivo seleccionado: {dispositivoSeleccionado}</h2>
	<button on:click={() => fetchData(dispositivoSeleccionado)}>Actualizar datos</button>
    <select bind:value={dispositivoSeleccionado} on:change={() => fetchRegistros()}>
      {#each dispositivos as dispositivo}
        <option value={dispositivo}>
          {dispositivo}
        </option>
      {/each}
    </select>

	<div id="tempDiv">
		<h3 id="tempInfo">Temperatura</h3>
		<h1 id="tempInfo">{datos[0].Temperatura_Celcius} ºC</h1>
	</div>

	{#if datos[0].Humedad != 0}
	<div id="humedadDiv">
		<h3 id="humedadInfo">Humedad</h3>
		<h1 id="humedadInfo">{datos[0].Humedad + "%"}</h1>
	</div>
	{/if}
    <h2>Registros históricos</h2>
    <table>
      <tr>
        {#each columns as column}
          <th>{column}</th>
        {/each}
      </tr>
      
      {#each datos as row}
        <tr>
          {#each Object.values(row) as cell}
		  {#if cell != 0}
            <td contenteditable="false">{cell}</td>
			{/if}
          {/each}
        </tr>
      {/each}
    </table>
    {/if}
  </div>
{/key}
  
</main>

<style>
	#tempDiv, #humedadDiv {
	  display: flex;
	  flex-direction: column;
	  justify-content: center;
	  margin: 2rem;
	}
	#tempInfo {
	  color: #0eafe9;
	  margin: 0;
	}

	main {
		background-color: #242424;
	}
	#humedadInfo {
	  color: rgb(28, 222, 35);
	  margin: 0;
	}
  
	main {
	  padding: 20px;
	  max-width: 800px;
	  margin: 0 auto;
	}
  
	.card {
	  border: 1px solid #ccc;
	  border-radius: 4px;
	  padding: 20px;
	  margin-bottom: 20px;
	}
  
	select {
	  padding: 8px;
	  font-size: 16px;
	  margin-bottom: 20px;
	}
  
	table {
	  width: 100%;
	  border: 1px solid #ccc;
	  border-collapse: collapse;
	}
  
	th, td {
	  border: 1px solid black;
	  padding: 8px;
	}

	th {
	  background-color: #4a4a4a;
	  width: 25%;
	}
  </style>