<template>
    <div :class="classes" ref="cell">
        <template v-if="renderType === 'index'">{{naturalIndex + 1}}</template>
        <template v-if="renderType === 'selection'">
            <Checkbox :value="checked" @click.native.stop="handleClick" @on-change="toggleSelect" :disabled="disabled"></Checkbox>{{naturalIndex + 1}}
        </template>
        <template v-if="renderType === 'selectlink'">
            <span>
                <Checkbox :value="checked" @click.native.stop="handleClick" @on-change="toggleSelect" :disabled="disabled"></Checkbox>
                <a @click="handleAClick" >{{naturalIndex + 1}}</a>
            </span>
        </template>
        <template v-if="renderType === 'html'"><span v-html="row[column.key]"></span></template>
        <template v-if="renderType === 'input'"><Input v-model="row[column.key]" :icon="column.icon" @on-click="colClick"></Input></template>
        <template v-if="renderType === 'select'">
            <Select  v-model="row[column.key+'-real']">
                    <Option v-for="opti in column.values" :value="opti.value" :key="opti.id">{{ opti.description }}</Option>
            </Select>
        </template>
        <template v-if="renderType === 'normal'"><span>{{row[column.key]}}</span></template>
        
        <template v-if="renderType === 'expand' && !row._disableExpand">
            <div :class="expandCls" @click="toggleExpand">
                <Icon type="ios-arrow-right"></Icon>
            </div>
        </template>
        <Cell
            v-if="renderType === 'render'"
            :row="row"
            :column="column"
            :index="index"
            :render="column.render"></Cell>
    </div>
</template>
<script>
    import Cell from './expand';
    import Icon from '../icon/icon.vue';
    import Checkbox from '../checkbox/checkbox.vue';
    import Input from '../input'

    export default {
        name: 'TableCell',
        components: { Icon, Checkbox, Cell },
        props: {
            prefixCls: String,
            row: Object,
            column: Object,
            naturalIndex: Number,    // index of rebuildData
            index: Number,           // _index of data
            checked: Boolean,
            disabled: Boolean,
            expanded: Boolean,
            fixed: {
                type: [Boolean, String],
                default: false
            }
        },
        data () {
            return {
                renderType: '',
                uid: -1,
                context: this.$parent.$parent.$parent.currentContext
            };
        },
        computed: {
            classes () {
                return [
                    `${this.prefixCls}-cell`,
                    {
                        [`${this.prefixCls}-hidden`]: !this.fixed && this.column.fixed && (this.column.fixed === 'left' || this.column.fixed === 'right'),
                        [`${this.prefixCls}-cell-ellipsis`]: this.column.ellipsis || false,
                        [`${this.prefixCls}-cell-with-expand`]: this.renderType === 'expand'
                    }
                ];
            },
            expandCls () {
                return [
                    `${this.prefixCls}-cell-expand`,
                    {
                        [`${this.prefixCls}-cell-expand-expanded`]: this.expanded
                    }
                ];
            }
        },
        methods: {
            colClick(){
                if(this.column.OnFKClick){
                    this.column.OnFKClick(this.row,this.index);
                }
            },
            toggleSelect () {
                this.$parent.$parent.$parent.toggleSelect(this.index);
            },
            toggleExpand () {
                this.$parent.$parent.$parent.toggleExpand(this.index);
            },
            handleClick () {
                // 放置 Checkbox 冒泡
            },
            handleAClick(){
                if(this.column.idRefScript){
                    this.column.idRefScript(this.row[this.column.key]);
                }
            }
        },
        created () {
            if (this.column.type === 'index') {
                this.renderType = 'index';
            } else if (this.column.type === 'selection') {
                this.renderType = 'selection';
            } else if (this.column.type === 'html') {
                this.renderType = 'html';
            } else if (this.column.type === 'expand') {
                this.renderType = 'expand';
            } else if (this.column.render) {
                this.renderType = 'render';
            } else if(this.column.type === 'input'){
                this.renderType = 'input';
            } else if(this.column.type === 'select'){
                this.renderType = 'select';
            }else if(this.column.type === 'selectlink'){
                this.renderType = 'selectlink';
            } else {
                this.renderType = 'normal';
            }
        }
    };
</script>
