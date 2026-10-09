<template>
    <KsNoData v-if="!variables.length" />

    <div v-else class="vars">
        <div class="vars-row vars-head">
            <KsText size="small">{{ $t(keyLabelTranslationKey) }}</KsText>
            <KsText size="small">{{ $t('value') }}</KsText>
        </div>

        <DynamicScroller
            :items="variables"
            :minItemSize="40"
            keyField="key"
            :buffer="200"
            :prerender="20"
            class="vars-rows"
        >
            <template #default="{item, index, active}">
                <DynamicScrollerItem
                    :item="item"
                    :active="active"
                    :dataIndex="index"
                >
                    <div class="vars-row">
                        <code class="vars-key">{{ item.key }}</code>

                        <div class="vars-value">
                            <KsDateAgo
                                v-if="item.date && typeof item.value === 'string'"
                                :inverted="true"
                                :date="item.value"
                            />
                            <template v-else-if="item.subflow && typeof item.value === 'string'">
                                {{ item.value }}
                                <SubFlowLink :executionId="item.value" />
                            </template>
                            <VarValue v-else :execution="executionsStore.execution" :value="item.value" :name="item.key" />
                        </div>
                    </div>
                </DynamicScrollerItem>
            </template>
        </DynamicScroller>
    </div>
</template>

<script setup lang="ts">
    import {computed} from "vue"
    import {DynamicScroller, DynamicScrollerItem} from "vue-virtual-scroller"
    import "vue-virtual-scroller/dist/vue-virtual-scroller.css"

    import * as Utils from "../../utils/utils"
    import VarValue from "./VarValue.vue"
    import SubFlowLink from "../flows/SubFlowLink.vue"
    import {useExecutionsStore} from "../../stores/executions"


    interface VariableRow {
        key: string;
        value: string | object | boolean | number;
        date?: boolean;
        subflow?: boolean;
    }

    const props = withDefaults(
        defineProps<{
            data: Record<string, unknown>;
            keyLabelTranslationKey?: string;
        }>(),
        {
            keyLabelTranslationKey: "name",
        },
    )

    const executionsStore = useExecutionsStore()

    const variables = computed<VariableRow[]>(() => {
        return Utils.executionVars(props.data)
    })
</script>
