<template>
	<div class="map">
		<div
				class="map-container"
				ref="svgMapContainer"
		>
			<mapScaleControlsComponent
					v-if="!hideScalePanel"
					class="map-container-scaleControls"
					:mapScale="mapScale"
					@scaleUp="scaleUp"
					@scaleDown="scaleDown"
					@scaleReset="resetScale"
			/>
			<mapMoveControlsComponent
					v-if="!hideMovePanel"
					class="map-container-moveControls"
					@moveUp="moveUp"
					@moveDown="moveDown"
					@moveLeft="moveLeft"
					@moveRight="moveRight"
					@moveCenter="calcCenter"
			/>
			<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
					:viewBox="wbStr"
					class="map-container-svg"
					@wheel.stop.prevent="whell"
					@mousedown.prevent="mouseDown"
					@mouseup.prevent="mouseup"
					@mouseleave.prevent="mouseleave"
					@mousemove.prevent="mousemove"
					@contextmenu.stop.prevent
					:class="{'map-container-svg-move': mouseDowned}"
					ref="svgMap"
			>
				<g ref="svgMapProp" v-html="svgString"></g>
				<g><slot></slot></g>
				<template v-if="mouseIsSelected">
					<polygon
							:points="selectionPoints"
							:stroke-width="widthLine*3"
							:stroke-dasharray="`${widthLine*2} ${widthLine*3}`"
							stroke="red"
							fill="none"
					/>
				</template>
			</svg>
		</div>
	</div>
</template>

<script>
import {throttle} from 'lodash';
import mapScaleControlsComponent from './components/mapScaleControlsComponent';
import mapMoveControlsComponent from './components/mapMoveControlsComponent';

let correctCoordinateDefault = 20;
let elementScaleDefault = 50;
let mapScaleDefault =1;
let throttleDelay = 100;


export default
{
	name: 'rnsvgmap',
	components: {mapScaleControlsComponent, mapMoveControlsComponent},
	props:
		{
			svg: String,
			maxCoords: Object,
			minCoords: Object,
			hideMovePanel: Boolean,
			hideScalePanel: Boolean
		},
		// ['svg', 'maxCoords', 'minCoords', 'hideMovePanel','hideScalePanel'],
	data: function()
	{
		return{
			svgMapCenter:{x:0,y:0},
			mapScale: mapScaleDefault,
			correctCoordinate: correctCoordinateDefault,
			elementScale: elementScaleDefault,
			mouseDowned: false,
			mouseIsSelected: false,
			startMove: {x:0,y:0},
			selection: {begin: {x:0,y:0}, end: {x:0,y:0}},
			tw: undefined,
			ts: undefined,
			mountedRefs: false,
			refs: undefined,
			mutation: undefined,
			svgSize: {x:297,y:210},
			svgString: ''
		}
	},
	created()
	{
		this.mountedRefs = false;
		this.refs = this.$refs;
		this.tw = throttle(this.mousemoveThrottle,throttleDelay);
		this.ts = throttle(this.mousemoveSelectionThrottle,throttleDelay);
		this.calcCenter();
	},
	destroyed()
	{
		if (this.mutation)
		{
			this.mutation.disconnect();
		}
	},
	mounted()
	{
		this.calcCenter();
		setTimeout(()=> {this.mountedRefs = true}, 1000);
	},
	computed:
	{
		/**
		 * расчёт коэфициента трансформвции
		 * между размерами контейнера и картинки svg
		 * */
		transformK()
		{
			if (!this.$refs.svgMap && !this.mountedRefs) return 1;
			let dMap = this.maxMap.y/this.maxMap.x;
			let dContainer = this.$refs.svgMap.height.baseVal.value/this.$refs.svgMap.width.baseVal.value;

			let dX = this.maxMap.x/this.$refs.svgMap.width.baseVal.value;
			let dY = this.maxMap.y/this.$refs.svgMap.height.baseVal.value;
			let d = (dMap > dContainer)? dY : dX;
			return d;
		},
		/**
		 * максимальные значения координат
		 * */
		max()
		{
			return {x: (this.maxCoords&&this.maxCoords.x)?this.maxCoords.x:this.svgSize.x, y: (this.maxCoords&&this.maxCoords.y)?this.maxCoords.y:this.svgSize.y};
		},
		min()
		{
			return {x: (this.minCoords&&this.minCoords.x)?this.minCoords.x:0, y: (this.minCoords&&this.minCoords.y)?this.minCoords.y:0};
		},
		/** утсановка размеров viewbox svg */
		wb()
		{
			// console.log('wb', this.testref);
			if (!this.$refs.svgMap && !this.mountedRefs)
			{
				return {
					x1: this.svgMapCenter.x - this.maxMap.x/(2),
					y1: this.svgMapCenter.y - this.maxMap.y/(2),
					x2: this.maxMap.x,
					y2: this.maxMap.y
				};
			}
			let dMap = this.maxMap.y/this.maxMap.x;
			let dContainer = this.$refs.svgMap.height.baseVal.value/this.$refs.svgMap.width.baseVal.value;
			let d = (dMap > dContainer);
			let maxX = d
				? this.$refs.svgMap.width.baseVal.value*this.transformK
				: this.maxMap.x;
			let maxY = d
				?  this.maxMap.y
				: this.$refs.svgMap.height.baseVal.value*this.transformK;
			let dd = {
				x1: this.svgMapCenter.x - (maxX/2),
				y1: this.svgMapCenter.y - (maxY/2),
				x2: maxX,
				y2: maxY
			};
			return dd;
		},
		wbStr()
		{
			return this.mountedRefs?`${this.wb.x1} ${this.wb.y1} ${this.wb.x2} ${this.wb.y2}`:'0 0 0 0';
		},
		maxMap()
		{
			return {x: Math.floor(this.dim.x/this.mapScale), y: Math.floor(this.dim.y/this.mapScale)};
		},
		dim()
		{
			return {x: this.max.x-this.min.x, y: this.max.y-this.min.y};
		},
		moveDim()
		{
			return Math.floor((Math.min(this.max.x, this.max.y)/10));
		},
		selectionPoints()
		{
			return `${this.selection.begin.x},${this.selection.begin.y} ${this.selection.end.x},${this.selection.begin.y} ${this.selection.end.x},${this.selection.end.y} ${this.selection.begin.x},${this.selection.end.y}`
		},
		widthLine()
		{
			return Math.min(this.max.x, this.max.y)/(200*this.mapScale);
		}
	},
	watch:
		{
			mountedRefs ()
			{
				this.redrawSvg(this.svg);
			},
			svg(val)
			{
				this.redrawSvg(val);
			}
		},
	methods:
	{
		redrawSvg(val)
		{
			if (val)
			{
				this.svgString = this.svg.trim()
					.replace(/<\?xml[\s\S]*?>/, '')
					.replace(/<!DOCTYPE[\s\S]*?>/, '')
					.replace(/<svg[\s\S]*?>/, '')
					.replace(/<\/svg>$/, '');
			}
		},
		calcCenter()
		{
			this.svgMapCenter.x = this.min.x+(this.max.x-this.min.x)/2;
			this.svgMapCenter.y = this.min.y+(this.max.y-this.min.y)/2;
		},
		scaleUp: function (e)
		{
			let mousePosition = (e)?this.getMouseMapCoord(e):undefined;
			this.mapScale++;
			if (e)
			{
				let mousePosition2 = this.getMouseMapCoord(e);
				this.svgMapCenter.x = this.svgMapCenter.x + (mousePosition.x - mousePosition2.x );
				this.svgMapCenter.y = this.svgMapCenter.y + (mousePosition.y - mousePosition2.y );
			}
		},
		scaleDown(e)
		{
			if (this.mapScale > 1)
			{
				let mousePosition = (e)?this.getMouseMapCoord(e):undefined;
				this.mapScale--;
				if (e)
				{
					let mousePosition2 = this.getMouseMapCoord(e);
					this.svgMapCenter.x = this.svgMapCenter.x + (mousePosition.x - mousePosition2.x );
					this.svgMapCenter.y = this.svgMapCenter.y + (mousePosition.y - mousePosition2.y );
				}
			}
		},
		moveUp()
		{
			this.svgMapCenter.y += this.moveDim/this.mapScale;
		},
		moveDown()
		{
			this.svgMapCenter.y -= this.moveDim/this.mapScale;
		},
		moveLeft()
		{
			this.svgMapCenter.x += this.moveDim/this.mapScale;
		},
		moveRight()
		{
			this.svgMapCenter.x -= this.moveDim/this.mapScale;
		},
		whell(e)
		{
			if (e.deltaY<0)
			{
				this.scaleUp(e);
			}
			else
			{
				this.scaleDown(e);
			}
		},
		mouseDown(e)
		{
			switch(e.button)
			{
				case 0:
					this.beginMove(e);
					break;
				case 1:
					this.setCenter(e);
					break;
				case 2:
					this.beginSelection(e);
					break;
			}
		},
		mouseup(e)
		{
			switch(e.button)
			{
				case 0:
					this.endMove();
					break;
				case 1:
					break;
				case 2:
					this.endSelection(e);
					break;
			}
		},
		mouseleave()
		{
			this.endMove();
		},
		mousemove(e)
		{
			if (this.mouseDowned)
			{
				this.tw(e);
			}
			if (this.mouseIsSelected)
			{
				this.ts(e);
			}
		},
		beginMove(e)
		{
			if (e && e.pageX && e.pageY)
			{
				this.mouseDowned = true;
				this.startMove.x = e.pageX;
				this.startMove.y = e.pageY;
			}
		},
		endMove()
		{
			this.mouseDowned = false;
			this.startMove.x = 0;
			this.startMove.y = 0;
		},
		mousemoveThrottle(e)
		{
			if (this.mouseDowned && this.startMove.x && this.startMove.y)
			{
				this.svgMapCenter.x -= this.transformK*(e.pageX-this.startMove.x);
				this.svgMapCenter.y -= this.transformK*(e.pageY-this.startMove.y);
				this.startMove.x = e.pageX;
				this.startMove.y = e.pageY;
			}
		},
		mousemoveSelectionThrottle(e)
		{
			if (this.mouseIsSelected)
			{
				this.selection.end = this.getMouseMapCoord(e);
			}
		},
		resetScale()
		{
			this.mapScale = 1;
		},
		setCenter(e)
		{
			this.svgMapCenter = this.getMouseMapCoord(e);
		},
		getMouseMapCoord(e)
		{
			return {x: this.wb.x1+e.offsetX*this.transformK, y: this.wb.y1+e.offsetY*this.transformK}
		},
		mutationCallback() // (mutationsList, observer)
		{
			// console.log(mutationsList);
		},
		beginSelection(e)
		{
			this.mouseIsSelected = true;
			this.selection.begin = this.getMouseMapCoord(e);
			this.selection.end = this.getMouseMapCoord(e);
		},
		endSelection(e)
		{
			try
			{
				if (this.mouseIsSelected)
				{
					this.selection.end = this.getMouseMapCoord(e);
					let dimX = this.selection.end.x - this.selection.begin.x;
					let dimY = this.selection.end.y - this.selection.begin.y;
					this.svgMapCenter.x = this.selection.begin.x + dimX/2;
					this.svgMapCenter.y = this.selection.begin.y + dimY/2;
					let dYX= dimY / dimX;
					let dMap = this.maxMap.y/this.maxMap.x;
					this.mapScale = Math.abs(Math.trunc((dYX > dMap)
						? this.dim.y / dimY
						: this.dim.x / dimX
					));
				}
			}
			finally
			{
				this.mouseIsSelected = false;
				this.selection.begin= {x:0,y:0};
				this.selection.end = {x:0,y:0};
			}
		},
	}
}

</script>

<style scoped lang="scss">

	.map
	{
		position: relative;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		padding: 0;
		margin: 0;

		&-container
		{
			padding: 0;
			margin: 0;
			position: relative;
			top: 0;
			left: 0;
			width: 100%;
			height: 100%;

			&-scaleControls
			{
				position: absolute;
				top: 20px;
				right: 40px;
				width: 0;
				height: 0;
			}

			&-moveControls
			{
				position: absolute;
				bottom: 80px;
				right: 40px;
				width: 0;
				height: 0;
			}

			&-svg
			{
				width: 100%;
				height: 100%;
				cursor: crosshair;
			}

			&-svg-move
			{
				cursor: move;
			}
		} // &-container
	} // .map
</style>
