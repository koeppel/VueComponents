<script setup lang="ts">
	import { VRow, VCol, VTextField, VSelect } from 'vuetify/lib/components/index.mjs';
	import { ref } from "vue";
	import KTimeSelect from "./KTimeSelect.vue";

	export interface Props {
		segment: TimerSegment,
		disabled: boolean
	}

	const props = defineProps<Props>();

	const innerSegment = ref<TimerSegment>(props.segment);

	const emit = defineEmits(["durationChanged", "nameChanged", "colorChanged"]);

	function setName(newName: string): void {
		innerSegment.value.name = newName;
		emit("nameChanged", newName);
	};

	function setDuration(duration: Duration): void {
		innerSegment.value.duration	= duration;
		emit("durationChanged", innerSegment.value.duration);
	};

	function setColor(color: string): void {
		innerSegment.value.pin.color = color;
		emit("colorChanged", color);
	};

</script>
<script lang="ts">
	import type { Duration } from "luxon";

	export type TimerSegmentPin = {
		color: string,
		rotation: number
	};

	export type TimerSegment = {
		name: string,
		duration: Duration,
		pin: TimerSegmentPin
	};
</script>
<template>
	<v-row style="align-items:center">
		<v-col cols="4">
			<slot name="name">
				<v-text-field
					v-model:model-value="innerSegment.name"
					@update:model-value="setName"></v-text-field>
				<v-select
					v-model:model-value="innerSegment.pin.color"
					:items="['Green', 'Yellow', 'Red', 'Grey']"
					:color="innerSegment.pin.color"
					variant="filled"
					@update:model-value="setColor"></v-select>
			</slot>
		</v-col>
		<v-col cols="8">
			<k-time-select
				:duration="innerSegment.duration"
				:disabled="disabled"
				@duration-changed="(duration) => { setDuration(duration); }">
				<template #prepend>
					<slot name="delete"></slot>
				</template>
			</k-time-select>
		</v-col>
	</v-row>
</template>