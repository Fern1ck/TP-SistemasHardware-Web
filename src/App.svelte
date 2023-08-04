<script lang="ts">
	import imgUrl from './circuitos.jpg';
	import { onMount } from 'svelte';
	let datos: { Fecha: string, Dispositivo: string, Datos: {Variable: string, Valor: string}[] }[];
	let dispositivos: string[];
	let dispositivoSeleccionado: string;
  
	async function fetchData(YaSeleccionado: string | null) {
	  //fetch dispositivos
	  const res = await fetch(`${import.meta.env.VITE_SERVERHOST}/dispositivos`);
	  dispositivos = await res.json();
	  //se inicializa el select al primer dispositivo
	  dispositivoSeleccionado = YaSeleccionado ?? dispositivos[0];
	  await fetchRegistros();
	}
  
	async function fetchRegistros() {
	  const res = await fetch(
		`${import.meta.env.VITE_SERVERHOST}/registros/${dispositivoSeleccionado}`
	  );
	  datos = await res.json();	 
	}
  
	async function descargarJson(){
		const res = await fetch(
		`${import.meta.env.VITE_SERVERHOST}/registrosJSON`
	  	);
		const blob = await res.blob()
 
    	var file = window.URL.createObjectURL(blob);

  		const downloadLink = document.createElement('a');
  		downloadLink.href = file;
  		downloadLink.download = 'registros.json'; 
			
  		downloadLink.click();
			
  		window.URL.revokeObjectURL(file);
  };
	
	onMount(() => {
	  document.body.style.backgroundImage = `url(${imgUrl})`;
	  fetchData(null)
	});
  
	let columns = [] ;
  
	$: {
	  if (datos) {
		  columns = [ ...Object.keys(datos[0])];
	  }
	}
  </script>
  
  <svelte:head>
	<title>TP Baldrich y Ramirez</title>
	<meta name="robots" content="noindex nofollow" />
	<html lang="en" />
  </svelte:head>
  
  <main>
	<h1>Sistemas de Hardware para la Administración</h1>
	<h3>Trabajo Práctico - Fernando Baldrich y Thiago Ramirez</h3>
  
	{#key datos}
	<div class="card">
	  {#if !datos}
	  	<h2 id="tempInfo">Cargando...</h2>
	  	{:else if datos.length === 0}
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
	  
		<div id="gridContainer">
			{#each datos[0].Datos as dato}
				<div class="gridItem">
					<h3 id="varName">{dato.Variable}</h3>
					<h1 id="varValue">{dato.Valor}</h1>
				</div>
			{/each}
		</div>
		<h4 id="varDate">Datos recolectados el {datos[0].Fecha}</h4>
		<button on:click={() => {descargarJson()}}>Descargar JSON</button>
	  	<h2>Registros históricos</h2>
	  	{#if datos.length > 0}
		  <table>
			<tr>
			  {#each columns as column}
				{#if column !== 'ID'} <!-- Exclude the 'ID' column -->
				  <th>{column}</th>
				{/if}
			  {/each}
			</tr>
		  
			{#each datos as row}
			  <tr>
				{#each Object.entries(row) as [key, value]}
				  {#if key !== 'ID'} <!-- Exclude the 'ID' column -->
					<td contenteditable="false">
					  {@html Array.isArray(value) ? value.map(item => `<u>${item.Variable}</u>: ${item.Valor}`).join('<br/>') : value}
					</td>
				  {/if}
				{/each}
			  </tr>
			{/each}
		  </table>
  		{/if}
	  {/if}
	</div>
	{/key}
  </main>

<style>
	* {
 		font-family: Arial;
		color: white;
	}
	#gridContainer{
	  justify-content: center;
	  display: grid;
  		grid-template-columns: 200px 200px 200px;
  		grid-gap: 30px;
	}
	#varValue {
	  color: #0eafe9;
	  margin: 0;
	}

	main {
		background-color: #242424;
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
	  color: black;
	}

	option {
		color: black;
	}
	button {
		color: black;
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