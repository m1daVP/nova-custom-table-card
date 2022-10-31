<template>
    <div class="overflow-x-auto bg-white dark:bg-gray-800 rounded-lg shadow h-full pt-4">
        <h1 v-if="title" class="h-6 flex mb-3 text-sm font-bold px-6">{{ title }}</h1>
        <table
            class="w-full"
            :class="card.style"
            data-testid="resource-table"
            ref="table"
        >
            <TableHeader
                :fields="header"
                :should-show-column-borders="shouldShowColumnBorders"
                :has-view-column="hasViewColumn"
            />
            <tbody>
                <TableRow v-for="(row, index) in rows"
                          :key="index"
                          :row="row"
                          :should-show-tight="shouldShowTight"
                          :should-show-column-borders="shouldShowColumnBorders"
                          :has-view-column="hasViewColumn"
                />
            </tbody>
        </table>
        <div v-if="viewAll && viewAll.label" class="w-full border-t border-gray-200 dark:border-gray-700 rounded-b-lg flex justify-center py-3">
            <div>
                <a class="text-primary-200 text-xs hover:text-primary-600" :href="viewAll.link">{{ viewAll.label }}</a>
            </div>
        </div>
        <pagination-links
            v-if="paginator && paginator.data > 0"
            :next="hasNextPage"
            :page="currentPage"
            :pages="totalPages"
            :per-page="perPage"
            :previous="hasPreviousPage"
        >
            <span
                v-if="resourceCountLabel"
                :class="{
                    'ml-auto': paginationComponent === 'pagination-links',
                }"
                class="text-sm text-80 px-4"
            >
                {{ resourceCountLabel }}
            </span>
        </pagination-links>
    </div>
</template>

<script>
import TableHeader from './TableHeader'
import TableRow from './TableRow'
import PaginationLinks  from './PaginationLinks'

export default {
    props: ['card'],

    components: {
        TableRow,
        TableHeader,
        PaginationLinks,
    },

    data() {
        return {
            rows: [],
            header: [],
            title: '',
            viewAll: false,
            paginator: {},
            perPage: 0,
            currentPage: 1,
            allMatchingResourceCount: 0,
        }
    },

    computed: {
        hasViewColumn() {
            return this.rows.find((row) => row.view)
        },

        shouldShowTight() {
            return this.card.style === 'table-tight'
        },

        /**
         * Determine if the resource table should show column borders.
         */
        shouldShowColumnBorders() {
            return !! this.card.showBorders
        },
        resourceCountLabel() {
            const first = this.perPage * (this.currentPage - 1);

            return (
                this.paginator.data.length &&
                `${first + 1}-${first + this.paginator.data.length} ${this.__('of')} ${
                    this.allMatchingResourceCount
                }`
            );
        },
        totalPages() {
            return Math.ceil(this.allMatchingResourceCount / this.perPage);
        },
        hasNextPage() {
            return this.paginator && this.paginator.next_page_url;
        },
        hasPreviousPage() {
            return this.paginator && this.paginator.prev_page_url;
        },
        shouldShowPagination() {
            return this.paginator &&
                this.paginator.data &&
                this.paginator.data.length > 0;
        },
    },

    mounted() {
        console.log(this.card)
        this.fillTableData(this.card)
        debugger;
        if (paginator !== undefined) {
            this.paginator = paginator
            this.perPage = paginator.per_page
            this.allMatchingResourceCount = paginator.total
        }

        // this.$refs['table'].parentNode.classList.remove('min-h-40')
    },

    methods: {
        fillTableData(card) {
            this.rows = card.rows
            this.header = card.header
            this.title = card.title
            this.viewAll = card.viewAll
        },
        selectPage(page) {
            this.currentPage = page;
            this.loadMore();
        },
        loadMore() {
            debugger;
            Nova.request()
                .get(`/nova-vendor/custom-table${this.config.routes[0].url}`, {
                    params: {
                        page: this.currentPage,
                    },
                })
                .then((response) => {
                    debugger;
                    this.paginator = response.data.paginator;
                    this.rows = response.data.rows;
                    if (this.paginator !== undefined) {
                        this.perPage = this.paginator.per_page;
                        this.allMatchingResourceCount = this.paginator.total;
                    }

                    Nova.$emit('resources-loaded');
                })
                .catch((error) => {
                    console.error(error);
                });
        },
    },

    watch: {
        card(value) {
            // Fix problem with dashboard caching 🤷‍♂️
            this.fillTableData(value)
        }
    }
}
</script>
