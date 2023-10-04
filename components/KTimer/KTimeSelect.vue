<script setup lang="ts">
	import { VContainer, VRow, VCol, VBtn } from 'vuetify/lib/components/index.mjs';
	import type { Duration, DurationLikeObject } from 'luxon';
	import { ref } from 'vue';

	export interface Props {
		duration: Duration,
		disabled: boolean
	};

	const props = withDefaults(defineProps<Props>(), {
		disabled: false
	});

	const innerDuration = ref<Duration>(props.duration);

	const emit = defineEmits(["durationChanged"]);

	function add(durationToAdd: DurationLikeObject): void {
		innerDuration.value = addDuration(innerDuration.value, durationToAdd);
		emit("durationChanged", innerDuration.value);
	};

	function subtract(durationToSubtract: DurationLikeObject):void {
		innerDuration.value = subtractDuration(innerDuration.value, durationToSubtract);
		emit("durationChanged", innerDuration.value);
	};

</script>
<script lang="ts">

	function addDuration(duration: Duration, durationToAdd: DurationLikeObject): Duration {
		const minutesToAdd = durationToAdd.minutes;
		const secondsToAdd = durationToAdd.seconds;

		if (secondsToAdd && duration.seconds === 59) {
			const addHour = duration.minutes === 59;

			duration = duration.set({
				hours: addHour ? duration.hours + 1 : duration.hours,
				minutes: addHour ? 0 : duration.minutes + 1,
				seconds: 0
			});
		} else if (minutesToAdd && duration.minutes === 59) {
			duration = duration.set({
				hours: duration.hours + 1,
				minutes: 0
			});
		} else {
			duration = duration.plus(durationToAdd);
		}

		return duration;
	};

	function subtractDuration(duration: Duration, durationToSubtract: DurationLikeObject): Duration {
		const minutesToSubstract = durationToSubtract.minutes;
		const secondsToSubstract = durationToSubtract.seconds;

		if (secondsToSubstract && duration.seconds === 0) {
			const subtractHour = duration.minutes === 0;

			duration = duration.set({
				hours: subtractHour ? duration.hours - 1 : duration.hours,
				minutes: subtractHour ? 59 : duration.minutes - 1,
				seconds: 59
			});
		} else if (minutesToSubstract && duration.minutes === 0) {
			duration = duration.set({
				hours: duration.hours - 1,
				minutes: 59
			});
		} else {
			duration = duration.minus(durationToSubtract);
		}

		return duration;
	};

	export { addDuration, subtractDuration };

</script>
<template>
	<v-container fluid class="px-0">
		<v-row class="justify-content-center px-0" style="align-items:center">
			<v-col cols="3" class="px-0">
				<v-container fluid class="text-center px-0">
					<v-row class="px-0">
						<v-col class="px-0">
							<v-btn
								icon="mdi-arrow-up"
								:disabled="disabled"
								@click="add({ hours: 1})"></v-btn>
						</v-col>
					</v-row>
					<v-row class="px-0">
						<v-col class="px-0">
							{{ duration.toFormat("hh") }}
						</v-col>
					</v-row>
					<v-row class="px-0">
						<v-col class="px-0">
							<v-btn
								icon="mdi-arrow-down"
								:disabled="disabled || innerDuration.hours === 0"
								@click="subtract({ hours: 1 })"></v-btn>
						</v-col>
					</v-row>
				</v-container>
			</v-col>
			<v-col cols="1" class="d-flex px-0" style="align-items: center; justify-content: center;">:</v-col>
			<v-col cols="3" class="px-0">
				<v-container fluid class="text-center px-0">
					<v-row class="px-0">
						<v-col class="px-0">
							<v-btn
								icon="mdi-arrow-up"
								:disabled="disabled"
								@click="add({ minutes: 1})"></v-btn>
						</v-col>
					</v-row>
					<v-row class="px-0">
						<v-col class="px-0">
							{{ duration.set({ hours: 0 }).toFormat("mm") }}
						</v-col>
					</v-row>
					<v-row class="px-0">
						<v-col class="px-0">
							<v-btn
								icon="mdi-arrow-down"
								:disabled="disabled || (innerDuration.hours === 0 && innerDuration.minutes === 0)"
								@click="subtract({ minutes: 1})"></v-btn>
						</v-col>
					</v-row>
				</v-container>
			</v-col>
			<v-col cols="1" class="d-flex px-0" style="align-items: center; justify-content: center;">:</v-col>
			<v-col cols="3" class="px-0">
				<v-container fluid class="text-center px-0">
					<v-row class="px-0">
						<v-col class="px-0">
							<v-btn
								icon="mdi-arrow-up"
								:disabled="disabled"
								@click="add({ seconds: 1})"></v-btn>
						</v-col>
					</v-row>
					<v-row class="px-0">
						<v-col class="px-0">
							{{ duration.set({ hours: 0, minutes: 0 }).toFormat("ss") }}
						</v-col>
					</v-row>
					<v-row class="px-0">
						<v-col class="px-0">
							<v-btn
								icon="mdi-arrow-down"
								:disabled="disabled || (innerDuration.hours === 0 && innerDuration.minutes === 0 && innerDuration.seconds === 0)"
								@click="subtract({ seconds: 1})"></v-btn>
						</v-col>
					</v-row>
				</v-container>
			</v-col>
			<v-col cols="1" class="px-0">
				<slot name="prepend"></slot>
			</v-col>
		</v-row>
	</v-container>
</template>