<script setup lang="ts">
	import { VContainer, VRow, VCol, VBtn } from 'vuetify/lib/components/index.mjs';
	import { onMounted, ref } from 'vue';
	import { Duration, DurationLikeObject } from "luxon";
	import KTimerSegment from'./KTimerSegment.vue';
	import type { TimerSegment } from './KTimerSegment.vue';
	import { subtractDuration } from './KTimeSelect.vue';

	export interface Props {
		segments?: TimerSegment[],
		editable?: boolean
	};

	const props = withDefaults(defineProps<Props>(), {
		segments: () => [],
		editable: false
	});

	let interval: number,
		secondsPassed: number = 0,
		segmentCache: TimerSegment[] = [];

	const timerStarted = ref<boolean>(false),
		timerStopped = ref<boolean>(false),
		segmentsEditable = ref<boolean>(props.editable);

	function startTimer(): void {
		timerStarted.value = true;
		timerStopped.value = false;
		segmentsEditable.value = false;
		currentSegment.value = getCurrentSegment();

		if (!segmentCache.length) segmentCache = innerSegments.value.map((segment) => { return {... segment} });

		interval = setInterval(() => tick({ seconds: 1 }), 1000);
	};

	function tick(durationToSubtract: DurationLikeObject):void {
		currentSegment.value = getCurrentSegment();

		if (currentSegment.value) {
			currentSegment.value.duration = subtractDuration(currentSegment.value.duration, durationToSubtract);
			secondsPassed++;
			updateTotalDuration();
			updateSpinnerRotation();
		}

		if (getSecondsOfDuration(totalDuration.value) === 0) stopTimer();
	};

	function stopTimer(): void {
		timerStopped.value = true;
		clearInterval(interval);
	};

	function resetTimer(): void {
		timerStopped.value = false;
		timerStarted.value = false;

		innerSegments.value.forEach((segment, index) => {
			segment.duration = segmentCache[index]?.duration;
		});

		segmentCache = [];
		secondsPassed = 0;
		updateTotalDuration();
		updateSpinnerRotation();

		if (props.editable) segmentsEditable.value = true;
	};

	const rotation = ref<string>('transform: rotate(0deg)');

	function updateSpinnerRotation(): void {
		const totalSeconds = getSecondsOfDuration(totalDuration.value.plus({ seconds: secondsPassed }));
		const degrees = totalSeconds > 0 ? (360 / totalSeconds) * secondsPassed : 0;

		rotation.value = `transform: rotate(${degrees}deg);`;
	};

	const totalDuration = ref<Duration>(Duration.fromObject({}));

	function updateTotalDuration(): void {
		const hours = getSumOfArray(innerSegments.value
			.map((segment: TimerSegment) => segment.duration.hours));

		const minutes = getSumOfArray(innerSegments.value
			.map((segment: TimerSegment) => segment.duration.minutes));

		const seconds = getSumOfArray(innerSegments.value
			.map((segment: TimerSegment) => segment.duration.seconds));

		totalDuration.value = totalDuration.value.set({
			hours: hours,
			minutes: minutes,
			seconds: seconds
		});
	};

	const innerSegments = ref<TimerSegment[]>(props.segments);

	const defaultSegment = { pin: { rotation: 0, color: "Grey" }, duration: Duration.fromObject({ hours: 0, minutes: 0, seconds: 0 }), name: ""};

	function addNewSegment(): void {
		innerSegments.value?.push({
			...defaultSegment,
			pin: {
				color: "Grey",
				rotation: 0
			},
			name: "New Timer Segment"
		});
	};

	function deleteSegment(segment: TimerSegment): void {
		innerSegments.value.splice(innerSegments.value.indexOf(segment), 1);
		updateSegments.value++;
		updateTotalDuration();
		updatePinRotations();
	};

	function onSegmentDurationChanged(segment: TimerSegment, duration: Duration): void {
		segment.duration = duration;
		updateTotalDuration();
		updatePinRotations();
	};

	function onPinColorChanged(segment: TimerSegment, color: string): void {
		segment.pin.color = color;
	};

	function updatePinRotations(): void {
		innerSegments.value.reduce((previousSegment: TimerSegment, segment: TimerSegment): TimerSegment => {
			const segmentSeconds = getSecondsOfDuration(segment.duration);
			const totalSeconds = getSecondsOfDuration(totalDuration.value.plus({ seconds: secondsPassed }));
			const degrees = totalSeconds > 0 ? (360 / totalSeconds) * segmentSeconds : 0;
			segment.pin.rotation = degrees + previousSegment.pin.rotation;

			return segment;
		}, { ...defaultSegment });
	};

	const currentSegment = ref<TimerSegment|undefined>(getCurrentSegment());

	function getCurrentSegment(): TimerSegment | undefined {
		return innerSegments.value.find((segment) => {
			return segment.duration.hours > 0
				|| segment.duration.minutes > 0
				|| segment.duration.seconds > 0;
		});
	};

	function getSumOfArray(array: number[]): number {
		return array.reduce((partialSum: number, a: number) => partialSum + a, 0);
	};

	function getSecondsOfDuration(duration: Duration): number {
		return duration.hours * 3600 + duration.minutes * 60 + duration.seconds;
	};

	function moveUp(segment: TimerSegment): void {
		move(segment, -1);
	};

	function moveDown(segment: TimerSegment): void {
		move(segment, 1);
	};

	const updateSegments = ref<number>(0);

	function move(segment: TimerSegment, moveBy: number): void {
		const tempSegments = [ ...innerSegments.value];
		const initialIndex = innerSegments.value.indexOf(segment);

		tempSegments[initialIndex + moveBy] = segment;
		tempSegments[initialIndex] = innerSegments.value[initialIndex + moveBy];

		innerSegments.value = tempSegments;
		updateSegments.value++;
	};

	onMounted(() => {
		updateTotalDuration();
	});

</script>
<template>
	<v-container fluid>
		<v-row>
			<v-col cols="12">
				<v-container fluid>
					<v-row>
						<v-col cols="12" class="d-flex justify-center">
							<div class="timerClock">
								<div id="timerSpinner" class="timerSpinner" :style="rotation">
									<div class="timerArrow"></div>
								</div>
								<div class="timerSegments">
									<div class="timerSegment" v-for="(segment, index) in innerSegments" :key="index + '-segment'" :style="'top: -' + index * 100 + '%;' + 'transform: rotate(' + segment.pin.rotation + 'deg);'">
										<div class="timerSegmentPin" :style="'border-color: ' + segment.pin.color + ';'"></div>
									</div>
								</div>
							</div>
						</v-col>
					</v-row>
					<v-row class="text-center" v-if="!segmentsEditable">
						<v-col cols="6">
							Total time remaining:<br/>{{ totalDuration.toFormat("hh:mm:ss") }}
						</v-col>
						<v-col cols="6" v-if="currentSegment">
							Current segment: {{ currentSegment.name }}<br/>
							{{ currentSegment.duration.toFormat("hh:mm:ss") }}
						</v-col>
					</v-row>
					<v-row justify="center">
						<v-col class="d-flex justify-center">
							<v-btn
								icon="mdi-play"
								:disabled="(timerStarted && !timerStopped) || getSecondsOfDuration(totalDuration) === 0"
								@click="startTimer"></v-btn>
						</v-col>
						<v-col class="d-flex justify-center">
							<v-btn
								icon="mdi-stop"
								:disabled="!timerStarted !== timerStopped"
								@click="stopTimer">
							</v-btn>
						</v-col>
						<v-col class="d-flex justify-center">
							<v-btn
								icon="mdi-refresh"
								:disabled="!timerStarted || !timerStopped"
								@click="resetTimer"></v-btn>
						</v-col>
					</v-row>
				</v-container>
			</v-col>
			<v-col cols="12" v-if="segmentsEditable">
				<v-container fluid>
					<v-row>
						<v-col cols="3">
							TotalTime:
						</v-col>
						<v-col cols="6">
							{{ totalDuration.toFormat("hh:mm:ss") }}
						</v-col>
					</v-row>
					<k-timer-segment v-for="(segment, index) in innerSegments" :key="index+updateSegments" :segment="segment" :disabled="!segmentsEditable"
						@duration-changed="(duration: Duration) => onSegmentDurationChanged(segment, duration)"
						@pin-color-changed="(color: string) => onPinColorChanged(segment, color)">
						<template #settings>
							<v-container fluid class="text-center">
								<v-row>
									<v-col>
										<v-btn icon="mdi-menu-up"
											:disabled="index === 0"
											@click="moveUp(segment)"></v-btn>
									</v-col>
								</v-row>
								<v-row>
									<v-col>
										<v-btn icon="mdi-delete"
											@click="deleteSegment(segment)"></v-btn>
									</v-col>
								</v-row>
								<v-row>
									<v-col>
										<v-btn icon="mdi-menu-down"
											:disabled="index === innerSegments.length - 1"
											@click="moveDown(segment)"></v-btn>
									</v-col>
								</v-row>
							</v-container>
						</template>
					</k-timer-segment>
					<v-row>
						<v-col cols="12" class="d-flex justify-center">
							<v-btn
								icon="mdi-plus"
								:disabled="!segmentsEditable"
								@click="addNewSegment"></v-btn>
						</v-col>
					</v-row>
				</v-container>
			</v-col>
		</v-row>
	</v-container>
</template>
<style>

	.timerClock {
		height: 364px;
		width: 364px;
		border: 2px solid black;
		border-radius: 100%;
		overflow: hidden;
	}

	.timerSpinner {
		width: 100%;
		height: 100%;
		position: relative;
		z-index: 2;
		transition-property: transform;
		transition-duration: 1s;
	}

	.timerArrow {
		display: block;
		height: 50%;
		border: 1px solid black;
		width: 0;
		position: relative;
		left: calc(50% - 1px);
	}

	.timerArrow::before {
		content: "";
		display: block;
		width: 0px;
		transform: rotate(45deg) translate(-10px, 5px);
		border: 1px solid black;
		height: 25px;
	}

	.timerArrow::after {
		content: "";
		display: block;
		width: 0px;
		transform: rotate(-45deg) translate(26px, -15px);
		border: 1px solid black;
		height: 25px;
	}

	.timerSegments {
		width: 100%;
		height: 100%;
		position: relative;
		top: -100%;
	}

	.timerSegment {
		position: relative;
		width: 100%;
		height: 100%;
	}

	.timerSegmentPin {
		display: block;
		height: 5%;
		border: 2px solid;
		border-color: grey;
		width: 0;
		position: relative;
		left: calc(50% - 2px);
	}

</style>