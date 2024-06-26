<script lang="ts">
import { createEventDispatcher } from 'svelte';

import SimpleColumn from './SimpleColumn.svelte';
import { Column, Row } from './table';
import Table from './Table.svelte';

let table: Table;
let selectedItemsNumber: number;

const dispatch = createEventDispatcher<{ update: string }>();

type Person = {
  id: number;
  name: string;
  age: number;
  hobby: string;
};

const people: Person[] = [
  { id: 1, name: 'John', age: 57, hobby: 'Skydiving' },
  { id: 2, name: 'Henry', age: 27, hobby: 'Cooking' },
  { id: 3, name: 'Charlie', age: 43, hobby: 'Biking' },
];

const idCol: Column<Person, string> = new Column('Id', {
  align: 'right',
  renderMapping: obj => obj.id.toString(),
  renderer: SimpleColumn,
  comparator: (a, b) => a.id - b.id,
});

const nameCol: Column<Person, string> = new Column('Name', {
  width: '3fr',
  renderMapping: obj => obj.name,
  renderer: SimpleColumn,
  comparator: (a, b) => a.name.localeCompare(b.name),
});

const ageCol: Column<Person, string> = new Column('Age', {
  align: 'right',
  renderMapping: obj => obj.age.toString(),
  renderer: SimpleColumn,
  comparator: (a, b): number => {
    dispatch('update', 'sorting');
    return a.age - b.age;
  },
  initialOrder: 'descending',
  overflow: true,
});

const hobbyCol: Column<Person, string> = new Column('Hobby', {
  renderMapping: obj => obj.hobby,
  renderer: SimpleColumn,
});

const columns: Column<Person, string>[] = [idCol, nameCol, ageCol, hobbyCol];
const selectable = (person: Person): boolean => person.age < 50;
const row = new Row<Person>({
  selectable,
  disabledText: 'People over 50 cannot be deleted',
});
</script>

<Table
  kind="people"
  bind:this="{table}"
  bind:selectedItemsNumber="{selectedItemsNumber}"
  data="{people}"
  columns="{columns}"
  row="{row}"
  defaultSortColumn="Id"
  on:update>
</Table>
