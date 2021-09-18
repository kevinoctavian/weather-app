<script>
	import { onMount } from "svelte";
	import { blur, fade } from "svelte/transition"
	import { Container, Row, Col, Image, Card, CardBody, CardSubtitle, CardText, Spinner  } from "sveltestrap";

	document.body.style = "background-color: rgba(240,240,240, 0.8)"

	let isCelcius = true;

	let coor = {lat: null, lon: null};
	let isLoading = false;
	let weather = null;
	
	function changeTemp() {
		isCelcius = !isCelcius;
	}

	onMount(() => {
		navigator.geolocation.getCurrentPosition(onSuccess, (e) => {alert("Jika ingin melihat data cuaca kamu harus klik izinkan")}, {enableHighAccuracy:true});
	});

	async function onSuccess(pos) {
		coor = {lat: pos.coords.latitude, lon: pos.coords.longitude};
		
		isLoading = true;

		if (!weather) {
			
			await fetch(
				`https://api.weatherapi.com/v1/forecast.json?key=kepo deh&q=${pos.coords.latitude},${pos.coords.longitude}&days=10&aqi=no&alerts=yes`
			)
			.catch(console.log)
			.then(res => res.json())
			.then((data) => {
				weather = data;
				isLoading = false;
			});
		}

	}

</script>

{#if weather}
<div in:fade>
	<Container class="mt-5">
			<Row class="mb-3">
				<Col md={12}>
					<Container fluid class="flex-column justify-content-center">
						<Row>
							<Col xs={12}>
								<div class="d-flex flex-column align-items-center weather-city">
									<Image src={weather.current.condition.icon} alt={weather.current.condition.text} width={60} />
									<h4>{weather.current.condition.text}</h4>
									<p class="text-center">
										{weather.location.name}, {weather.location.region}, {weather.location.country}
										<br />
										<small>
											<b>{weather.location.tz_id}</b>
										</small>
									</p>
									<p>
										{#if weather.current.is_day === 1}
											Day
										{:else}
											Night
										{/if}
									</p>
									<p>{weather.current.last_updated}</p>
								</div>
							</Col>
						</Row>

						<Row>
							<Col xs={12}>
								<div class="d-flex flex-column align-items-center temperature" on:click={changeTemp} >
									<div class="d-flex temp">
										{#if isCelcius}
											<h3 class="me-2" transition:blur={{amount: 10}}>{Math.round(weather.current.temp_c)}°C</h3>
										{:else}
											<h3 class="me-2" transition:blur={{amount: 10}}>{Math.round(weather.current.temp_f)}°F</h3>
										{/if}
									</div>
									
									<div class="d-flex flex-column feels-like">
										

										<small class="me-2 position-relative">
											{#if isCelcius}
											feels like: {Math.round(weather.current.feelslike_c)}°C
											{:else}
											feels like: {Math.round(weather.current.feelslike_f)}°F
											{/if}
										</small>
									</div>
								</div>
							</Col>
						</Row>
						
					</Container>

				</Col>

				<Col md={12} class="weather-status">
					<Container fluid>
						<Row class="text-center my-4">
							<Col class="mb-2 mx-2 bg-light shadow">
								<p class="mb-0">Humidity</p>
								<h5>{weather.current.humidity} %</h5>
							</Col>

							<Col class="mb-2 mx-2 bg-light shadow">
								<p class="mb-0">Pressure</p>
								<h5>{weather.current.pressure_mb} millibar</h5>
							</Col>

							<Col class="mb-2 mx-2 bg-light shadow">
								<p class="mb-0">Cloud</p>
								<h5>{weather.current.cloud} %</h5>
							</Col>

							<Col class="mb-2 mx-2 bg-light shadow">
								<p class="mb-0">Wind Speed</p>
								<h5>{weather.current.wind_kph} Km/H</h5>
							</Col>

							<Col class="mb-2 mx-2 bg-light shadow">
								<p class="mb-0">Wind gust</p>
								<h5>{weather.current.gust_kph} Km/H</h5>
							</Col>

							<Col class="mb-2 mx-2 bg-light shadow">
								<p class="mb-0">Precipitation</p>
								<h5>{weather.current.precip_mm} mm</h5>
							</Col>
						</Row>
					</Container>
				</Col>

			</Row>

			<h4 class="my-3">Days</h4>
			<Row class="weather-hourly">
				{#each weather.forecast.forecastday as forecast}
					<Col>
						<Card>
							<CardBody class="d-flex flex-column align-items-center">
								<CardSubtitle class="mb-2 text-center">{forecast.date}</CardSubtitle>
								<img class="img-fluid" src={forecast.day.condition.icon} alt={forecast.day.condition.text}>
								<CardText class="text-center">
									{#if isCelcius}
										<h3 class="me-2" transition:blur={{amount: 10}}>{Math.round(forecast.day.avgtemp_c)}°C</h3>
									{:else}
										<h3 class="me-2" transition:blur={{amount: 10}}>{Math.round(forecast.day.avgtemp_f)}°F</h3>
									{/if}
								</CardText>
								<CardText class="text-center">
									{forecast.day.condition.text}
								</CardText>
								<CardText class="text-center mt-2">
									Change Of Rain {forecast.day.daily_chance_of_rain} %
								</CardText>
							</CardBody>
						</Card>
					</Col>
				{/each}
			</Row>

			<h4 class="my-3">Hourly</h4>
			<Row class="weather-hourly mb-4">
				{#each weather.forecast.forecastday as forecast}
					{#each forecast.hour as hour}
						<Col>
							<Card>
								<CardBody>
									<CardSubtitle class="mb-2 text-center">{hour.time}</CardSubtitle>
									<img class="img-fluid" src={hour.condition.icon} alt={hour.condition.text}>
									<CardText class="text-center">
										{#if isCelcius}
											<h3 class="me-2" transition:blur={{amount: 10}}>{Math.round(hour.temp_c)}°C</h3>
										{:else}
											<h3 class="me-2" transition:blur={{amount: 10}}>{Math.round(hour.temp_f)}°F</h3>
										{/if}
									</CardText>
									<CardText class="text-center mb-auto">
										{hour.condition.text}
									</CardText>
									<CardText class="text-center mt-2">
										<small>
											Change Of Rain {hour.chance_of_rain} %
										</small>
									</CardText>
								</CardBody>
							</Card>
						</Col>
					{/each}
				{/each}
			</Row>

			<Row class="justify-content-center">
				<p class="text-center" >Powered by <a href="https://www.weatherapi.com/" title="Free Weather API">WeatherAPI.com</a></p>
			</Row>

	</Container>
</div>
{:else}
	{#if isLoading}
		<h1 class="text-center">Tunggu sebentar sedang fetch data</h1>
		<div class="d-flex justify-content-center">
			<Spinner secondary />
		</div>
	{:else}
		<h1 class="text-center">Mohon Izinkan mengakses lokasi untuk mendapatkan informasi cuaca di daerah kamu</h1>
	{/if}

{/if}

<style>
:global(body) {
    height: 100%;
    margin: 0;
    padding: 0;
}

:global(.weather-status h5) {
	font-weight: bold;
}

:global(.weather-hourly) {
	width: 100%;
	flex-direction: row;
	flex-wrap: nowrap;
	overflow-x: auto;
	align-self: stretch;
}

:global(.weather-hourly .card) {
	height: 100%;
}

:global(.temp) {
	cursor: pointer;
}

@media screen and (max-width: 768px) {
	:global(.weather-status p) {
		font-size: 0.9rem;
	}
	:global(.weather-status h5) {
		font-size: 0.8rem;
	}
}
</style>