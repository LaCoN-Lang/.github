> ⚠️ **Early development:** This project, being experimental, has only recently come into existence and still has yet to be fully implemented.

## The Programming Language

**LaCoN** is an embeddable, gradual typing, multi-paradigm, data-and-scripting-oriented language designed to bridge the gap between structured information and executable logic.<br>
It can be used to represent static data as well as dynamic data with both full and restricted logic execution.

<br>
<details>
  <summary>Русский</summary>
  
  > ⚠️ **Ранняя стадия разработки:** Этот проект как экспериментальный начал своё существования совсем недавно и ему ещё предстоит быть полноценно реализованным.

**LaCoN** — встраиваемый, мультипарадигменный, ориентированный на данные и сценарии язык с последовательной типизацией.<br>
Он может быть использован для представления статических данных и динамических данных с полноценным или ограниченным исполнением логики.
</details>

## Idea Examples
<details>
  <summary>Static, Dynamic data and Logic</summary>

**File:** `energy_generator.slacon`<br>
Clear data example of static lacon file used as schema.

```ts
entity-type: energy-generator // In Static LaCoN strings do not require quotes
entity-name: <String> // Explicit type annotation used
entity-description?: <String>
energy<Dictionary>: {
	// Using inbuilt physical units types
	// Unicode symbols are allowed, × === *, ÷ === /
	power-reproduction<Expr>: <Power> × delta <Time>
	fuel<Dictionary>: {
		// Restricted set of allowed values specified
		type: <String> of ("solid" | "liquid" | "gas") 
		consumption<Expr>: <Mass> ÷ delta <Time>
		// “?” marks optional keys
    // <Dictionary[]> marks an Array of Dictionaries
		emission<Dictionary[]>?: [
			// “*” marks repeatable elements
			{compound: <String>, per-consumption: <Mass>}*
		]
	}
	heat-generation<Expr>?: <Temperature> ÷ delta <Time>
}
craft<Dictionary[]>: [
	{component: <String>, count: <Int>}*
]
craft-time: <Time>
cost<Dictionary>: {requisition: <Number>, energy: <Number>}
size<Dictionary>: {L: <Length>, W: <Length>, H: <Length>}
weight: <Mass>
durability: <Number>
```
**File**: `energy_generators.llacon`<br>
This file consumes `schema` to build a list of generators.

```ts
// Marks output type as Array of Dictionaries
[Marker:Output<Dictionary[]>]
// Using of `schema`, if output is an Array, then schema applies to each element in the Array
// Schema limits allowed data in each element
use schema './energy_generator.slacon'
{
	entity-type: energy-generator
	entity-name: solar-generator-mk1
	entity-description: Базовый солнечный генератор для малых станций
	energy: {
		power-reproduction: 50W × 1min
		heat-generation: 1°C ÷ 1min
	}
	craft: [
		{component: metal-plate, count: 10},
		{component: solar-cell, count: 8},
		{component: circuit-board, count: 5}
	]
	craft-time: 30s
	cost: {requisition: 200, energy: 50}
	size: {L: 2m, W: 1m, H: 1m}
	weight: 150kg
	durability: 100
}
{
	entity-type: energy-generator
	entity-name: coal-generator
	entity-description: Угольный генератор средней мощности для промышленного использования
	energy: {
		power-reproduction: 200W × 1min
		fuel: {
			type: solid
			consumption: 5kg ÷ 1min
			emission: [
				{compound: CO2, per-consumption: 4.5kg},
				{compound: SO2, per-consumption: 0.5kg}
			]
		}
		heat-generation: 150°C ÷ 1min
	}
	craft: [
		{component: steel-plate, count: 20},
		{component: combustion-chamber, count: 1},
		{component: coolant-pipe, count: 5}
	]
	craft-time: 120s
	cost: {requisition: 1000, energy: 200}
	size: {L: 5m, W: 2m, H: 2m}
	weight: 1200kg
	durability: 500
}
{
	entity-type: energy-generator
	entity-name: steam-generator
	entity-description: Паровой генератор средней мощности для промышленных и энергетических нужд
	energy: {
		power-reproduction: 150W × 1min
		fuel: {
			type: liquid
			consumption: 3L ÷ 1min
			emission: [
				{compound: H2O, per-consumption: 3L}
			]
		}
		heat-generation: 120°C ÷ 1min
	}
	craft: [
		{component: steel-plate, count: 15},
		{component: boiler, count: 1},
		{component: pipe, count: 10},
		{component: valve, count: 4}
	]
	craft-time: 90s
	cost: {requisition: 700, energy: 150}
	size: {L: 4m, W: 2m, H: 2m}
	weight: 900kg
	durability: 400
}
// Spread and some other directives (only with yield) to dynamically generate data in “.llacon” files
spread(["steam-generator-mk2", "steam-generator-mk3", "steam-generator-mk4", "steam-generator-mk5"], [{ ... }, { ... }, { ... }, { ... }]) as (generator, data){
	yield {
		entity-type: energy-generator
		entity-name: ${generator}
		entity-description: ${data.description}
		energy: {
			type: liquid
			consumption: ${data.consumption}
			emission: ${data.emission}
		}
		...data
	}
}
```
**File**: `energy_generators.lacon`<br>
The endpoint uses a `.lacon` file with full access to executable logic to read a `.llacon` file built according to the `.slacon` schema rules, and registers our objects in the hypothetical system.
```ts
const generatorsData = readData("./energy_generators.llacon")

public function registerGenerator(generator<Dictionary>) {
	GameAPI.registerEntity({
		type: generator["entity-type"],
		name: generator["entity-name"],
		description: generator["entity-description"] ?? "Описание отсутствует",
		energy: generator.energy,
		craft: generator.craft,
		craftTime: generator["craft-time"],
		cost: generator.cost,
		size: generator.size,
		weight: generator.weight,
		durability: generator.durability
	})
}

for gen in generatorsData {
	registerGenerator(gen)
}
```

**Note:** this structure is by no means mandatory; a single `.lacon` file can handle this task on its own.

</details>
