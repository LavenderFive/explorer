<script lang="ts" setup>
import TwoFactorAuthDialog from '@/plugins/vuetify/@core/components/TwoFactorAuthDialog.vue';
import { PageRequest } from '@/types';
import type { PropType } from 'vue';
import { computed, ref } from 'vue';

const props = defineProps({
    total: { type: Number },
    limit: { type: Number },
    callback: { type: Function, required: true },
});
const current = ref(1)
const showSize = 3
const pages = computed(() => {
    const pages: {color: string, page: number}[] = [] 
    if(props.total && props.limit && props.total > props.limit) {
        let page = 0
        while(true) {
            if( page * props.limit > props.total ) break
            page += 1
            if( props.total / props.limit > 10 && page > showSize && page < (props.total / props.limit - showSize + 1)) {
                if(!(page >= current.value - 1 && page <= current.value + 1)){
                    continue
                }
            } 
            pages.push({
                color: page === current.value? 'btn-primary': '',
                page: page,
            })
        }
    }
    return pages
})

function gotoPage(pageNum: number) {
    current.value = pageNum
    props.callback(pageNum)
}

</script>
<template>
    <div class="my-5 text-center">
        <div v-if="total && limit" class="btn-group">
            <button v-for="{page, color} in pages" class="btn btn-md" :class="color" @click="gotoPage(page)">{{ page }}</button>
        </div>
    </div>
</template>
