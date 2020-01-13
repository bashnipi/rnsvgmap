<template>
	<div class="map-controls-component">
		<div class="map-controls-component-move">
			<button class="map-controls-component-button" data-button-move="up" @click="moveUp">&#9650;</button>
			<button class="map-controls-component-button" data-button-move="left" @click="moveLeft">&#9668;</button>
			<button class="map-controls-component-button" data-button-move="center" @click="moveCenter">&#9900;</button>
			<button class="map-controls-component-button" data-button-move="right" @click="moveRight">&#9658;</button>
			<button class="map-controls-component-button" data-button-move="down" @click="moveDown">&#9660;</button>
		</div>
	</div>
</template>

<script>

export default
{
	name: 'mapMoveControlsComponent',
	props:['mapScale'],
	methods:
	{
		moveUp()
		{
			this.$emit('moveUp');
		},
		moveDown()
		{
			this.$emit('moveDown');
		},
		moveLeft()
		{
			this.$emit('moveLeft');
		},
		moveRight()
		{
			this.$emit('moveRight');
		},
		moveCenter()
		{
			this.$emit('moveCenter');
		}
	}
}

</script>

<style scoped lang="scss">

	$button-size: 20px;
	.map-controls-component
	{
		position:relative;
		padding: 0;
		margin: 0;
		user-select: none;

		&-button
		{
			width: $button-size;
			height: $button-size;
			padding: 0;
			text-align: center;
			vertical-align: middle;
			border: 1px solid #EF6B01;
			background-color: white;
			color: #EF6B01;
		}

		&-move
		{
			position: relative;
			/*margin-top:20px;*/

			button[data-button-move="up"]
			{
				position: absolute;
				top: 0;
				left: 0;
			}
			button[data-button-move="left"]
			{
				position: absolute;
				top: $button-size;
				left: -$button-size;
			}
			button[data-button-move="right"]
			{
				position: absolute;
				top: $button-size;
				left: $button-size;
			}
			button[data-button-move="center"]
			{
				position: absolute;
				top: 20px;
				left: 0;
				border: none;
			}
			button[data-button-move="down"]
			{
				position: absolute;
				top: 40px;
				left: 0;
			}
		}

	}

</style>
