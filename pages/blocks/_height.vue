<script>
    import {getBlock, getTransactionList} from "~/api";
    import {getTimeDistance, getTimeUTC} from "~/assets/utils";
    import TransactionList from '~/components/TransactionList';
    import BackButton from '~/components/BackButton';

    export default {
        components: {
            TransactionList,
            BackButton,
        },
        asyncData({ params, error }) {
            return getBlock(params.height)
                .then((block) => {
                    return {
                        block : {
                            ...block,
                            timeDistance: getTimeDistance(block.timestamp),
                            timeUTC: getTimeUTC(block.timestamp),
                        }
                    };
                })
                .catch((e) => {
                    error({ statusCode: 404, message: 'Block not found' });
                });
        },
        data() {
            return {
                /** @type Block */
                block: {},
                //isTxListLoading: true,
                txList: [],
            }
        },
        computed: {
            prevUrl() {
                return this.block.height > 1 ? '/blocks/' + (this.block.height - 1) : false;
            },
            nextUrl() {
                return this.block.height < this.block.latestBlockHeight ? '/blocks/' + (this.block.height + 1) : '';
            },
        },
        mounted() {
            getTransactionList({block: this.block.height})
                .then((txListInfo) => {
                    if (txListInfo.data && txListInfo.data.length) {
                        this.txList = txListInfo.data;
                    }
                });
        }
    }
</script>

<template>
    <div>
        <section class="panel u-section">
            <div class="panel__section panel__header">
                <h1 class="panel__header-title panel__title">
                    <BackButton/>
                    Block Information
                </h1>
            </div>
            <dl>
                <dt>Height</dt>
                <dd>{{ block.height }}</dd>

                <dt>TimeStamp</dt>
                <dd>{{ block.timeDistance }} ago ({{ block.timeUTC }})</dd>

                <dt>Hash</dt>
                <dd>{{ block.hash }}</dd>

                <dt>Validator</dt>
                <dd>
                    <nuxt-link class="link--default" :to="'/address/' + block.validators[0].address" v-if="block.validators[0]">
                        {{ block.validators[0].address }}
                    </nuxt-link>
                </dd>

                <dt>Size</dt>
                <dd>{{ block.size }} bytes</dd>

                <dt>Reward</dt>
                <dd>{{ block.reward }} {{ $store.state.COIN_NAME }}</dd>
            </dl>
        </section>
        <div class="u-section navigation">
            <nuxt-link class="button button--ghost-main" :to="prevUrl" v-if="prevUrl">Prev Block</nuxt-link>
            <nuxt-link class="button button--ghost-main" :to="nextUrl" v-if="nextUrl">Next Block</nuxt-link>
        </div>
        <section class="panel u-section" v-if="txList.length">
            <div class="panel__section panel__header">
                <h2 class="panel__header-title panel__title">
                    <img class="panel__header-title-icon" src="/img/icon-transaction.svg" alt="" role="presentation">
                    Transactions
                </h2>
            </div>
            <TransactionList :tx-list="txList" :current-block="block.height"/>
        </section>
    </div>
</template>