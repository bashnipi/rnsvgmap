<template>
	<div class="map-controls-component">
		<div class="map-controls-component-scale">
			<button class="map-controls-component-button" @click="scaleUp">+</button>
			<button class="map-controls-component-button" @click="scaleReset" data-button-scale="reset">1:1</button>
			<button class="map-controls-component-button" @click="scaleDown">-</button>
			<div>{{mapScale}}</div>
		</div>
	</div>
</template>

<script>

export default
{
	name: 'mapScaleControlsComponent',
	props:['mapScale'],
	methods:
	{
		scaleUp: function ()
		{
			this.$emit('scaleUp');
		},
		scaleDown()
		{
			this.$emit('scaleDown');
		},
		scaleReset()
		{
			this.$emit('scaleReset');
		}
	}
}

</script>

<style>

	.map-controls-component
	{
		position: relative;
		padding: 0;
		margin: 0;
		user-select: none;
	}
	.map-controls-component-button
	{
		width: 20px;
		height: 20px;
		padding: 0;
		margin: 0;
		text-align: center;
		vertical-align: middle;
		border: 1px solid #EF6B01;
		background-color: white;
		/*width: 23px;*/
		color: #EF6B01;
	}

	.map-controls-component-scale
	{
		position: relative;
		margin-top: 20px;
	}
	button[data-button-scale="reset"]
	{
		border-bottom: none;
		border-top: none;
	}

</style>
