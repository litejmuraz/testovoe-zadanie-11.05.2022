<script>
	let months = {
		1: "Янв.",
		2: "Фев.",
		3: "Март",
		4: "Апр.",
		5: "Май",
		6: "Июнь",
		7: "Июль",
		8: "Авг.",
		9: "Сен.",
		10: "Окт.",
		11: "Ноя.",
		12: "Дек."
	};
	let months_full = {
		1: "Январь",
		2: "Февраль",
		3: "Март",
		4: "Апрель",
		5: "Май",
		6: "Июнь",
		7: "Июль",
		8: "Август",
		9: "Сентябрь",
		10: "Октябрь",
		11: "Ноябрь",
		12: "Декабрь"
	};
	let weekdays = {
		1: "Понедельник",
		2: "Вторник",
		3: "Среда",
		4: "Четверг",
		5: "Пятница",
		6: "Суббота",
		7: "Воскресенье"
	};
	
	let legend = [
		{type: 1, text: "No", key: "L1"},
		{type: 2, text: "1-9", key: "L2"},
		{type: 3, text: "10-19", key: "L3"},
		{type: 4, text: "20-29", key: "L4"},
		{type: 5, text: "30+", key: "L5"}
	];

	function sortObjectByKeys(o) {
		return Object.keys(o).sort().reduce((r, k) => (r[k] = o[k], r), {});
	}

	function num0(v) {
		return v < 10 ? ("0" + v) : v;
	}

	let data = [];

	let tw;

	let selected_date = "";

	function select(date) {
		selected_date = date;
	}

	fetch('https://test.digitalpartnersglobal.com/test/calendar.json')
		.then(response => {
			response.json().then(contributions => {
				data = [];

				let date = new Date();
				let wd = date.getDay();
				if (wd == 0) wd = 7;
				wd--;
				date.setDate(date.getDate() - wd + 7);

				for (let w = 1; w <= 51; w++) {
					let days = [];

					let key = "";
					let month = 0;

					for(let d = 7; d >= 1; d--) {
						date.setDate(date.getDate() - 1);
						if (d == 4) {
							month = date.getMonth() + 1;
							key = date.getFullYear() + "_" + (month < 10 ? ("0" + month) : month);
						}

						let s_date = date.getFullYear() + "-" + num0(date.getMonth() + 1) + "-" + num0(date.getDate());
						let v = contributions.hasOwnProperty(s_date) ? contributions[s_date] : 0;

						let type = 0;;
						if (v == 0) type = 1;
						if (v >= 1 && v <= 9) type = 2;
						if (v >= 10 && v <= 19) type = 3;
						if (v >= 20 && v <= 29) type = 4;
						if (v >= 30) type = 5;

						days.unshift({
							wd: d,
							date: s_date,
							date_display: months_full[date.getMonth() + 1] + " " + date.getDate() + ", " + date.getFullYear(),
							v: v,
							type: type
						});
					}

					if (!data.hasOwnProperty(key)) data[key] = {name: months[month], weeks: []};
					data[key].weeks.unshift({days: days});
				}

				data = Object.values(sortObjectByKeys(data));
			});
		});
</script>

<br><br><br>

<div class="months">
	{#each data as month}
		<div class="month-name" style="width: {17 * month.weeks.length}px">{month.name}</div>
	{/each}
</div>

<div class="weeks">
	<div class="week">
		<div class="week-name">Пн</div>
		<div class="week-name"></div>
		<div class="week-name">Ср</div>
		<div class="week-name"></div>
		<div class="week-name">Пт</div>
		<div class="week-name"></div>
		<div class="week-name"></div>
	</div>
	{#each data as month}
		{#each month.weeks as week}
			<div class="week">
				{#each week.days as day}
					<div class="day con{day.type} {day.date == selected_date ? 'selected' : ''}" on:click="{e => select(day.date)}">
						{#if day.date == selected_date}
							<div class="tooltip" style="left: {-tw/2+7.5}px" bind:clientWidth={tw}>
								<div>{day.v > 0 ? day.v : 'No'} contributions</div>
								{weekdays[day.wd]}, {day.date_display}
							</div>
						{/if}
					</div>
				{/each}
			</div>
		{/each}
	{/each}
</div>

<div class="legend">
	<div class="legend-text">Меньше</div>
	{#each legend as legend_item}
		<div class="day con{legend_item.type} {legend_item.key == selected_date ? 'selected' : ''}" on:click="{e => select(legend_item.key)}">
			{#if legend_item.key == selected_date}
				<div class="tooltip" style="left: {-tw/2+7.5}px" bind:clientWidth={tw}>
					<div>{legend_item.text} contributions</div>
				</div>
			{/if}
		</div>
	{/each}
	<div class="legend-text">Больше</div>
</div>

<style>
	.legend {
		display: flex;
		flex-direction: row;
		margin-top: 15px;
	}
	.legend-text {
		text-align: center;
		color: #959494;
		font-size: 12px;
		line-height: 15px;
		width: 100px;
	}

	.months {
		display: flex;
		flex-direction: row;
		padding-left: 25px;
	}
	.month-name {
		text-align: center;
		color: #959494;
		font-size: 12px;
		line-height: 15px;
		margin-bottom: 5px;
	}

	.weeks {
		display: flex;
		flex-direction: row;
	}
	.week {
		display: flex;
		flex-direction: column;
	}
	.week-name {
		margin: 1px;
		width: 25px;
		height: 15px;
		font-size: 12px;
		line-height: 15px;
		color: #959494;
	}

	.day {
		position: relative;
		margin: 1px;
		width: 15px;
		height: 15px;
	}
	.day:hover {
		margin: 0;
		border: solid 1px rgba(0, 0, 0, 0.5);
	}
	.day.selected {
		margin: 0;
		border: solid 1px rgba(0, 0, 0, 0.9);
	}

	.con1 {
		background: #EDEDED;
	}
	.con2 {
		background: #ACD5F2;
	}
	.con3 {
		background: #7FA8C9;
	}
	.con4 {
		background: #527BA0;
	}
	.con5 {
		background: #254E77;
	}
	
	.tooltip {
		background-color: #000000;
		border-radius: 3px;
		position: absolute;
		bottom: 28px;
		color: #7C7C7C;
		padding: 5px 9px;
		text-align: center;
		z-index: 10;
		white-space: nowrap;
	}
	.tooltip div {
		color: #FFFFFF;
		display: block;
	}
	.tooltip::after {
		content: '';
		display: inline-block;
		position: absolute;
		left: calc(50% - 10px);
		top: 100%;
		border: 10px solid transparent;
		border-top: 10px solid #000000;
	}
</style>
