<script setup lang="ts">
	import { VContainer, VRow, VCol, VBtn } from 'vuetify/lib/components/index.mjs';
	import { ref, Ref } from 'vue';
	import { DateTime } from "luxon";

	export interface Props {
		birthdays?: BirthDayWrapper[]
	}

	const props = defineProps<Props>();

	const selectedDate: Ref<DateTime> = ref<DateTime>(DateTime.now());

	const isBirthDay = function(date: DateTime):boolean | undefined {
		return props.birthdays?.some((birthdayObject: BirthDayWrapper) => {
			let birthday = DateTime.fromFormat(birthdayObject.date, "yyyy-MM-dd");
			return birthday.day === date.day && birthday.month === date.month;
		});
	}

	const isCurrentDay = function(date: DateTime):boolean {
		return today.year === date.year
			&& today.month === date.month
			&& today.day === date.day;
	}

	const getCSSClassesForDateTime = function(dateTime: DateTime, month: number):string {
		let classes = "text-center";

		if (dateTime.month !== month) {
			classes += " notCurrentMonth"
		}

		if (isBirthDay(dateTime)) {
			classes += " text-green"
		}

		return classes;
	}

	const getKeyForWeek = function(week: Week):string {
		let weekDateParts: string[] = week.days[0].toISOWeekDate().split("-");
		return weekDateParts[0] + "-" + weekDateParts[1];
	}

</script>
<script lang="ts">
	export const today: DateTime = DateTime.now()

	// TODO: i18n this
	export const weekDays: string[] = [
		"Mo", "Tu", "We", "Th", "Fr", "Sa", "Su"
	]

	export type Week = {
		days: DateTime[],
		month: number,
		year: number
	}

	export type BirthDayWrapper = {
		name: string,
		date: string
	}

	export function getWeeksForMonth(month: number, year: number): Week[] {
		let startingDayOfWeek: number = 1; // 1 = Monday
		let amountOfWeeks: number = 6;
		let weeks: Week[] = [];
		let firstDayOfMonth: DateTime = DateTime.now().set({ day: 1, month: month, year: year});

		while(firstDayOfMonth.weekday !== startingDayOfWeek) {
			firstDayOfMonth = firstDayOfMonth.minus({ days: 1 });
		}

		for (let weekCount = 0 ; weekCount < amountOfWeeks ; weekCount++) {
			let week: Week = {
				days: [],
				month: month,
				year: year
			};

			for (let day = 0 ; day < 7 ; day++) {
				let date = firstDayOfMonth.plus({
					days: (7 * weekCount) + day
				});
				week.days.push(date);
			}

			weeks.push(week);
		}

		return weeks;
	}

</script>
<template>

	<div>
		<v-container>
			<v-row>
				<v-col class="text-center">
					<v-btn
						icon="mdi-minus"
						@click="selectedDate = selectedDate.minus({ years: 1})"></v-btn>
					<v-btn>
						{{  selectedDate.year }}
					</v-btn>
					<v-btn
						icon="mdi-plus"
						@click="selectedDate = selectedDate.plus({ years: 1})"></v-btn>
				</v-col>
				<v-col class="text-center">
					<v-btn
						icon="mdi-minus"
						@click="selectedDate = selectedDate.minus({ months: 1})"></v-btn>
					<v-btn>
						{{  selectedDate.monthShort }}
					</v-btn>
					<v-btn
						icon="mdi-plus"
						@click="selectedDate = selectedDate.plus({ months: 1})"></v-btn>
				</v-col>
			</v-row>
			<v-row>
				<v-col class="text-center">
					KW
				</v-col>
				<v-col class="text-center" v-for="weekDay in weekDays" :key="weekDay">
					{{ weekDay }}
				</v-col>
			</v-row>
			<v-row v-for="week in getWeeksForMonth(selectedDate.month, selectedDate.year)" :key="getKeyForWeek(week)">
				<v-col class="text-center">
					{{ week.days[0].toISOWeekDate().split("-")[1] }}
				</v-col>
				<v-col v-for="day in week.days" :class="getCSSClassesForDateTime(day, selectedDate.month)" :key="day.toISO()">
					<v-btn
						@click="selectedDate = day"
						:active="isCurrentDay(day)"
						variant="text"
						stacked>
						<template v-slot:append>
							<v-icon v-if="isBirthDay(day)">mdi-cake-variant</v-icon>
						</template>
						{{ day.day }}
					</v-btn>
				</v-col>
			</v-row>
		</v-container>
	</div>

</template>
<style scoped>
	.notCurrentMonth {
		opacity: 50%;
	}

	.isBirthday {
		color: green !important;
	}
</style>