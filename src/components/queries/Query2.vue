<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
    <div>
        <v-dialog v-model="dialog" max-width="600" persistent>
            <template v-slot:activator="{ on }">
                <v-btn fab dark color="indigo" v-on="on" fixed right bottom>
                    <v-icon dark>search</v-icon>
                </v-btn>
            </template>
            <v-card>
                <v-card-title>
                    <span class="headline">Запит №2</span>

                    <v-tooltip bottom>
                        <template v-slot:activator="{ on }">
                            <v-btn color="primary" dark v-on="on">
                                <v-icon
                                        small
                                        class="mr-2"
                                        @click="editItem(props.item)"
                                >
                                    info
                                </v-icon>
                            </v-btn>
                        </template>
                        <span> Одержати кількість та перелік виробів окремої категорії та загалом, які були виготовлені вказаним цехом, ділянкою, підприємством загалом за вказаний проміжок часу.</span>
                    </v-tooltip>

                </v-card-title>

                <v-card-text>
                    <v-container grid-list-md>
                        <v-layout wrap>
                            <v-flex xs12 sm12 md12>
                                <v-select
                                        :items="tables"
                                        v-model="currentTable"
                                        item-text="name"
                                        return-object
                                        label="Категорія виробів"
                                ></v-select>
                            </v-flex>
                            <v-flex xs12 sm12 md12>
                                <v-select
                                        :items="areas"
                                        v-model="currentArea"
                                        item-text="name"
                                        item-value="id"
                                        return-object
                                        label="Ділянка"
                                ></v-select>
                            </v-flex>
                            <v-flex xs12 sm12 md12>
                                <v-select
                                        :items="departments"
                                        v-model="currentDepartment"
                                        item-text="address"
                                        item-value="id"
                                        return-object
                                        label="Цех"
                                ></v-select>
                            </v-flex>
                            <v-flex xs12 sm12 md12>
                                <v-select
                                        :items="enterprises"
                                        v-model="currentEnterprise"
                                        item-text="name"
                                        item-value="id"
                                        return-object
                                        label="Підприємство"
                                ></v-select>
                            </v-flex>

                            <v-flex md6>
                                <v-menu
                                        min-width="240px"
                                        :close-on-content-click="false"
                                        :nudge-right="40"
                                        v-model="datePicker1"
                                >
                                    <v-text-field
                                            :value="startDate"
                                            slot="activator"
                                            label="Початкова дата"
                                            prepend-icon="date_range"
                                    ></v-text-field>
                                    <v-date-picker
                                            v-model="startDate"
                                            no-title
                                            scrollable
                                    >
                                        <v-spacer></v-spacer>
                                        <v-btn flat color="primary" @click="datePicker1 = false">OK</v-btn>
                                    </v-date-picker>
                                </v-menu>
                            </v-flex>
                            <v-flex md6>
                                <v-menu
                                        min-width="240px"
                                        :close-on-content-click="false"
                                        :nudge-right="40"
                                        v-model="datePicker2"
                                >
                                    <v-text-field
                                            :value="endDate"
                                            slot="activator"
                                            label="Початкова дата"
                                            prepend-icon="date_range"
                                    ></v-text-field>
                                    <v-date-picker
                                            v-model="endDate"
                                            no-title
                                            scrollable
                                    >
                                        <v-spacer></v-spacer>
                                        <v-btn flat color="primary" @click="datePicker2 = false">OK</v-btn>
                                    </v-date-picker>
                                </v-menu>
                            </v-flex>
                        </v-layout>
                    </v-container>
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" flat @click="closeForm">Відмінити</v-btn>
                    <v-btn color="blue darken-1" flat @click="search">Пошук</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <universal-table
                :api-link="apiLink"
                :items="items"
                :headers="headers"
                :loading="loading"
                table-name="Запит 2"
                :hidden-action="true"
        ></universal-table>
    </div>
</template>

<script>
    import {HTTP} from "@/util/HTTP";
    import UniversalTable from "@/components/UniversalTable";

    export default {
        name: "Query2",
        components: {UniversalTable},
        data() {
            return {
                /*props for table*/
                items: [],
                headers: [
                    {text: 'Назва виробу', value: 'name', sortable: false},
                ],
                loading: false,

                /*props for dialog and others*/
                apiLink: '',
                loadAll: false,
                dialog: true,

                /*objects for select*/
                tables: [
                    {name: 'Планери', apiLink: 'gliders', queryLink: 'get-ready-by-area-enterprise-department'},
                    {name: 'Дельтаплани', apiLink: 'hang_gliders', queryLink: 'get-ready-by-area-enterprise-department'},
                    {name: 'Гелікоптери', apiLink: 'helicopters', queryLink: 'get-ready-by-area-enterprise-department'},
                    {name: 'Літаки', apiLink: 'planes', queryLink: 'get-ready-by-area-enterprise-department'},
                    {name: 'Ракети', apiLink: 'rockets', queryLink: 'get-ready-by-area-enterprise-department'},
                    {name: 'Усі категорії'},
                ],
                departments: [],
                areas: [],
                enterprises: [],

                /*save current select value*/
                currentArea: '',
                currentTable: {},
                currentDepartment: '',
                currentEnterprise: {},

                /*fields for date*/
                startDate: '',
                endDate: '',
                datePicker1: false,
                datePicker2: false,
            }
        },
        watch: {
            currentTable() {
                this.loadAll = !this.currentTable.apiLink;
            }
        },
        methods: {
            closeForm() {
                this.dialog = false;
            },
            async search() {
                this.items = [];
                this.loading = true;
                if (this.loadAll) {
                    for (let table of this.tables) {
                        if (table.apiLink) {
                            try {
                                const { data } = await HTTP.get(`/api/${table.apiLink}/${table.queryLink}`, {
                                    params: {
                                        "department_id": this.currentDepartment.id,
                                        "area_id": this.currentArea.id,
                                        "enterprise_id": this.currentEnterprise.id,
                                        "start_date": this.startDate,
                                        "end_date": this.endDate,
                                    }
                                });
                                this.items.push(...data);
                            } catch (e) {
                                console.log(e);
                            } finally {
                                this.closeForm();
                                this.loading = false
                            }
                        }
                    }
                } else {
                    try {
                        const { data } = await HTTP.get(`/api/${this.currentTable.apiLink}/${this.currentTable.queryLink}`, {
                            params: {
                                "department_id": this.currentDepartment.id,
                                "area_id": this.currentArea.id,
                                "enterprise_id": this.currentEnterprise.id,
                                "start_date": this.startDate,
                                "end_date": this.endDate,
                            }
                        });
                        this.items = data;
                    } catch (e) {
                        console.log(e);
                    } finally {
                        this.closeForm();
                        this.loading = false
                    }
                }
            }
        },
        async mounted() {
            try {
                const { data } = await HTTP.get(`/api/enterprise`);
                this.enterprises = data;
            } catch (e) {
                console.log(e);
            }

            try {
                const { data } = await HTTP.get(`/api/departments`);
                this.departments = data;
            } catch (e) {
                console.log(e);
            }

            try {
                const { data } = await HTTP.get(`/api/areas`);
                this.areas = data;
            } catch (e) {
                console.log(e);
            }
        }
    }
</script>
