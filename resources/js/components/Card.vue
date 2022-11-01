<template>
    <div class="overflow-x-auto bg-white dark:bg-gray-800 rounded-lg shadow h-full pt-4">
        <h1 v-if="title" class="h-6 flex mb-3 text-sm font-bold px-6">{{ title }}</h1>
        <div
            v-if="isLoading"
            class="rounded-lg flex items-center justify-center absolute inset-0 z-30 pt-2"
        >
            <loader class="text-gray-300" width="50" />
        </div>
        <template v-else>
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
                v-if="paginator && paginator.data && paginator.data.length > 0"
                :next="hasNextPage"
                :page="currentPage"
                :pages="totalPages"
                :per-page="perPage"
                :previous="hasPreviousPage"
                @page="loadMore"
            >
                <span
                    v-if="resourceCountLabel"
                    class="text-sm text-80 px-4 ml-auto"
                >
                    {{ resourceCountLabel }}
                </span>
            </pagination-links>
        </template>
    </div>
</template>

<script>
import TableHeader from './TableHeader'
import TableRow from './TableRow'
import PaginationLinks  from './PaginationLinks'
import Loader from './Loader'

export default {
    props: ['card'],

    components: {
        TableRow,
        TableHeader,
        PaginationLinks,
        Loader,
    },

    data() {
        return {
            rows: [],
            header: [],
            title: '',
            viewAll: false,
            paginator: {},
            paginationUrl: '',
            perPage: 0,
            currentPage: 1,
            allMatchingResourceCount: 0,
            isLoading: false,
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
        this.fillTableData(this.card)
        if (this.card.paginator) {
            this.paginator = this.card.paginator
            this.perPage = this.card.paginator.per_page
            this.allMatchingResourceCount = this.card.paginator.total
        }
        if (this.card.paginationUrl) {
            this.paginationUrl = this.card.paginationUrl
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
        loadMore(page) {
            this.isLoading = true;
            Nova.request()
                .get(this.paginationUrl, {
                    params: {
                        page,
                        per_page: this.perPage,
                    },
                })
                .then((response) => {
                    this.currentPage = page;
                    this.paginator = response.data.paginator;
                    this.rows = response.data.rows;
                    if (this.paginator) {
                        this.perPage = this.paginator.per_page;
                        this.allMatchingResourceCount = this.paginator.total;
                    }
                })
                .catch((error) => {
                    Nova.error(`Loading error: ${error.message}`);
                    console.error(error);
                })
                .finally(() => {
                    this.isLoading = false;
                    Nova.$emit('resources-loaded');
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
