<script setup lang="ts">
	import { VContainer, VRow, VCol, VBtn } from 'vuetify/lib/components/index.mjs';
	import { ref } from 'vue';
	import type { TimerSegment } from './KTimerSegment.vue';
	import { Duration } from "luxon";
	import KTimerSegment from'./KTimerSegment.vue';
	import { subtractDuration } from './KTimeSelect.vue';

	const segments = ref<TimerSegment[]>([]);

	const defaultSegment = { pin: { rotation: 0, color: "Grey" }, duration: Duration.fromObject({ hours: 0, minutes: 0, seconds: 0 }), name: ""};

	function addNewSegment(): void {
		segments.value?.push({
			...defaultSegment,
			pin: {
				color: "Grey",
				rotation: 0
			},
			name: "New Timer Segment"
		});
	};

	function updateSegment(segment: TimerSegment, duration: Duration): void {
		segment.duration = duration;
		updateTotalTime();
		updatePinRotations();
	};

	function updatePinRotations(): void {
		segments.value.reduce((previousSegment: TimerSegment, segment: TimerSegment): TimerSegment => {
			const segmentSeconds = getSecondsOfDuration(segment.duration);
			const degrees = totalSeconds > 0 ? (360 / totalSeconds) * segmentSeconds : 0;
			segment.pin.rotation = degrees + previousSegment.pin.rotation;

			return segment;
		}, { ...defaultSegment });
	};

	function updateColor(segment: TimerSegment, color: string): void {
		segment.pin.color = color;
	};

	const totalTime = ref<TimerSegment>({
		...defaultSegment,
		name: "Total Time"
	});

	function updateTotalTime(): void {
		const hours = getSumOfArray(segments.value
			.map((segment: TimerSegment) => segment.duration.hours));

		const minutes = getSumOfArray(segments.value
			.map((segment: TimerSegment) => segment.duration.minutes));

		const seconds = getSumOfArray(segments.value
			.map((segment: TimerSegment) => segment.duration.seconds));

		totalTime.value.duration = totalTime.value.duration.set({
			hours: hours,
			minutes: minutes,
			seconds: seconds
		});

		if (!timerStarted.value) totalSeconds = getSecondsOfDuration(totalTime.value.duration);
	};

	function getSumOfArray(array: number[]): number {
		return array.reduce((partialSum: number, a: number) => partialSum + a, 0);
	};

	function deleteSegment(segment: TimerSegment): void {
		segments.value.splice(segments.value.indexOf(segment), 1);
		updateTotalTime();
		updatePinRotations();
	};

	let segmentCache: TimerSegment[] = [];

	let interval: number;

	let totalSeconds: number = 0,
		secondsPassed: number = 0;

	const disableSegments = ref<boolean>(false);

	const timerStarted = ref<boolean>(false);

	const timerStopped = ref<boolean>(false);

	function startTimer(): void {
		timerStarted.value = true;
		timerStopped.value = false;

		disableSegments.value = true;

		if (!segmentCache.length) segmentCache = segments.value.map((segment) => { return {... segment} });

		interval = setInterval(() => {
			const currentSegment = segments.value.find((segment) => {
				return segment.duration.hours > 0
					|| segment.duration.minutes > 0
					|| segment.duration.seconds > 0;
			});

			if (currentSegment) {
				currentSegment.duration = subtractDuration(currentSegment.duration, { seconds: 1 });
				updateTotalTime();
				secondsPassed++;
				updateSpinnerRotation();
			}

			if (totalTime.value.duration.hours === 0
				&& totalTime.value.duration.minutes === 0
				&& totalTime.value.duration.seconds === 0) stopTimer();
		}, 1000);
	};

	function stopTimer(): void {
		timerStopped.value = true;
		clearInterval(interval);
	};

	function resetTimer(): void {
		timerStopped.value = false;
		timerStarted.value = false;

		segments.value.forEach((segment, index) => {
			segment.duration = segmentCache[index]?.duration;
		});

		segmentCache = [];
		totalSeconds = 0;
		secondsPassed = 0;
		updateTotalTime();
		updateSpinnerRotation();

		disableSegments.value = false;
	};

	const rotation = ref<string>('transform: rotate(0deg)');

	function updateSpinnerRotation(): void {
		const degrees = totalSeconds > 0 ? (360 / totalSeconds) * secondsPassed : 0;

		rotation.value = `transform: rotate(${degrees}deg);`;
	};

	function getSecondsOfDuration(duration: Duration): number {
		return duration.hours * 3600 + duration.minutes * 60 + duration.seconds;
	};

</script>
<template>
	<v-container>
		<v-row>
			<v-col cols="12">
				<v-container>
					<v-row>
						<v-col cols="12" class="d-flex justify-center">
							<div class="timerClock">
								<div id="timerSpinner" class="timerSpinner" :style="rotation">
									<div class="timerArrow"></div>
								</div>
								<div class="timerSegments" v-if="segments.length > 1">
									<div class="timerSegment" v-for="(segment, index) in segments" :key="index + '-segment'" :style="'top: -' + index * 100 + '%;' + 'transform: rotate(' + segment.pin.rotation + 'deg);'">
										<div class="timerSegmentPin" :style="'border-color: ' + segment.pin.color + ';'"></div>
									</div>
								</div>
							</div>
						</v-col>
					</v-row>
					<v-row justify="center">
						<v-col class="d-flex justify-center">
							<v-btn
								icon="mdi-play"
								:disabled="timerStarted && !timerStopped"
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
			<v-col cols="12">
				<v-container>
					<v-row>
						<v-col cols="3">
							TotalTime:
						</v-col>
						<v-col cols="6">
							{{ totalTime.duration.toFormat("hh:mm:ss") }}
						</v-col>
					</v-row>
					<k-timer-segment v-for="(segment, index) in segments" :key="index" :segment="segment" :disabled="disableSegments"
						@duration-changed="(duration: Duration) => updateSegment(segment, duration)"
						@color-changed="(color: string) => updateColor(segment, color)">
						<template #delete>
							<v-btn icon="mdi-delete"
								@click="deleteSegment(segment)"></v-btn>
						</template>
					</k-timer-segment>
					<v-row>
						<v-col cols="12" class="d-flex justify-center">
							<v-btn
								icon="mdi-plus"
								:disabled="disableSegments"
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