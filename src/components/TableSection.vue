<template>
    <section>
        <div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 my-2">
            <h1 class="h3 text-uppercase font-weight-bolder"><span v-if="currentMaster">{{currentMaster.name}}</span></h1>
        </div>
        <b-row>
            <b-col lg="10" class="my-1">
                <b-form-group
                        label="Filter"
                        label-cols-sm="1"
                        label-align-sm="right"
                        size="sm"
                        label-for="filterInput"
                        class="mb-0"
                >
                    <b-input-group size="sm">
                        <b-form-input
                                v-model="filter"
                                type="search"
                                id="filterInput"
                                placeholder="Type to Search"
                        ></b-form-input>
                        <b-input-group-append>
                            <b-button :disabled="!filter" @click="filter = ''">Clear</b-button>
                        </b-input-group-append>
                    </b-input-group>
                </b-form-group>
            </b-col>

            <b-col lg="2" class="my-1" align="right">
                <b-dropdown right text="Show/Hide Columns" size="sm" align="right" >
                    <b-dropdown-form v-for="field in fields"
                                     :key="field.key">
                        <b-form-checkbox v-model="field.class" v-bind:value="'d-none'">{{ field.label }}</b-form-checkbox>
                    </b-dropdown-form>
                </b-dropdown>
            </b-col>

            <b-col lg="12" class="my-1">
                <b-form-group
                        label="Filter On"
                        label-cols-sm="1"
                        label-align-sm="right"
                        label-size="sm"
                        description="Leave all unchecked to filter on all data"
                        class="mb-0">
                    <b-form-checkbox-group v-model="filterOn" class="mt-1">
                        <b-form-checkbox
                                v-for="field in fields"
                                :key="field.key"
                                :value="field.key">
                            {{ field.label }}
                        </b-form-checkbox>
                    </b-form-checkbox-group>
                </b-form-group>
            </b-col>

            <b-col sm="5" md="6" class="my-1">
                <b-form-group
                        label="Per page"
                        label-cols-sm="6"
                        label-cols-md="4"
                        label-cols-lg="3"
                        label-align-sm="right"
                        label-size="sm"
                        label-for="perPageSelect"
                        class="mb-0"
                >
                    <b-form-select
                            v-model="perPage"
                            id="perPageSelect"
                            size="sm"
                            :options="pageOptions"
                    ></b-form-select>
                </b-form-group>
            </b-col>

            <b-col sm="7" md="6" class="my-1">
                <b-pagination
                        v-model="currentPage"
                        :total-rows="totalRows"
                        :per-page="perPage"
                        align="fill"
                        size="sm"
                        class="my-0"
                ></b-pagination>
            </b-col>

        </b-row>
        <div class="table-responsive">
            <b-table striped hover
                     :items="filtered_items"
                     :fields="fields"
                     :sort-by.sync="sortBy"
                     :sort-desc.sync="sortDesc"
                     :sort-direction="sortDirection"
                     :filter="filter"
                     :filterIncludedFields="filterOn"
                     :current-page="currentPage"
                     :per-page="perPage"
                     @filtered="onFiltered">
                <template v-slot:cell(task_percentage)="data">
                    <span class="text-monospace">{{ data.value.toFixed(2) }}%</span>
                </template>
            </b-table>
        </div>
    </section>
</template>

<script>
import master_json from "../data/masters";

export default {
	name: "TableSection",
	props: {
		configData: Object,
	},
	data() {
		return {
			mastersData: master_json,
		    currentMaster: null,
			monstersData: null,
			total_weight: 0,
			sortDirection: 'desc',
			sortDesc: true,
			sortBy: 'task_percentage',
			filter: null,
			filterOn: [],
			perPage: 15,
			pageOptions: [10, 15, 25],
			totalRows: 1,
			currentPage: 1,
			fields: [
				{
					key: 'id',
					label: 'Monster ID',
					sortable: true,
					class: 'd-none',
				},
				{
					key: 'monster',
					label: 'Monster name',
					sortable: true,
					class: '',
				},
				{
					key: 'combat_req',
					label: 'Combat Requirement',
					sortable: true,
					class: 'd-none',
				},
				{
					key: 'slayer_req',
					label: 'Slayer Requirement',
					sortable: true,
					class: 'd-none',
				},
				{
					key: 'defence_req',
					label: 'Defence Requirement',
					sortable: true,
					class: 'd-none',
				},
				{
					key: 'task_percentage',
					label: 'Task chance',
					sortable: true,
					class: '',
				},
			],
			filtered_items: [],
		}
	},
	methods: {
		reload() {
			//set current Slayer Master
		    this.currentMaster = this.mastersData[this.$route.params.id];
		    this.monstersData = this.currentMaster.assignments;

		    //filter the list of Monsters and calculate task chances
		    this.filterData();
			this.generateTaskWeights();
		},
	    filterData() {
			this.filtered_items = this.monstersData;
			let removeIds = [];

			//filter based on Stats
			this.configData.statUnlocks.forEach(stat=>{
				//for each stat requirement, remove those from list where the current stat value is greater or equal to the monsters requirement value.
				this.filtered_items = _.filter(this.filtered_items, function(monster){ return (parseInt(stat.value.current) >= parseInt(monster[stat.filter])) || stat.ignore === 'true' } );
			});

			//filter based on Point Unlocks
			this.configData.pointUnlocks.forEach(reward=>{
	            if (reward.unlock === 'true' && reward.block) {
					removeIds = removeIds.concat(reward.monster_ids)
				}
	            else if (reward.unlock === 'false' && !reward.block && reward.masters.includes(this.currentMaster.id)) {
					removeIds = removeIds.concat(reward.monster_ids)
				}
			});

			//filter based on Block List
			removeIds = removeIds.concat(this.configData.blockList.map(item => item.monster_ids));

			//filter based on Quest unlocks
			removeIds = removeIds.concat(this.configData.questUnlocks.filter(quest => quest.unlock === 'false').flatMap(item => item.monster_ids));

			//handle removal list
			this.filtered_items = _.filter(this.filtered_items, function(monster){ return !removeIds.includes(parseInt(monster.id)) } );

			//recalculate row length
			this.totalRows = this.filtered_items.length
		},
		generateTaskWeights() {
			//calculate total weight
			this.total_weight = this.filtered_items.reduce(function(prev, cur) {
				return prev + parseInt(cur.taskweight);
			}, 0);

			//add new entry with calculated task change
			this.filtered_items.forEach(item => {
				item.task_percentage = item.taskweight / this.total_weight * 100;
			})
		},
		onFiltered(filteredItems) {
			this.totalRows = filteredItems.length
			this.currentPage = 1
		},
	},
	created() {
		this.reload();
	},
	watch: {
		$route(to, from) {
			this.reload();
		}
	}
}
</script>

<style scoped>

</style>
