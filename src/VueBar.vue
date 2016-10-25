<template>
    <div @touchstart="_onTouchStart"
         @mousedown="_onTouchStart">
        <slot name="bar"></slot>
    </div>
</template>


/*!
* vue-bar v0.0.1 (https://github.com/johnnyGoo/vue-bar)
* Author: Johnny chen
*
* Copyright 2013-2016 Johnny chen
*/
<script>


    var count = 0;

    if (!window.Smart) {
        throw 'vue-bar required smart.js'
    }
    var Smart = window.Smart;
    var Css = Smart.Css;
    var _ = Smart._;

    // 注册
    export default {
        // 声明 props
        props: {
            onlyExternal: {
                type: Boolean,
                default: false
            },
            value: {
                type: Object,
                default: function () {
                    return {x: 0, y: 0}
                }
            },
            bound: {
                type: Object,
                default: function () {
                    return {minX: 0, minY: 0, maxX: 100, maxY: 0}
                }
            },
            scale: {
                type: Number,
                default: 1
            },
            time:{
                type: Number,
                default: 0.5
            }
        },
        data: function () {
            return {
                startPos: {pageX: 0, pageY: 0},
                startValue: {},
                display: false,
                dragging: false,

            }

        },
        watch: {
            'value.x+value.y': function () {
                this._matchToBound(this.value);
            }
        }
        ,
        methods: {
            _matchToBound(v){
                if (v.x < this.bound.minX) {
                    v.x = this.bound.minX
                } else if (v.x > this.bound.maxX) {
                    v.x = this.bound.maxX
                }
                if (v.y < this.bound.minY) {
                    v.y = this.bound.minY
                } else if (v.y > this.bound.maxY) {
                    v.y = this.bound.maxY
                }
                v.percentX = (v.x - this.bound.minX) / (this.bound.maxX - this.bound.minX) || 0;
                v.percentY = (v.y - this.bound.minY) / (this.bound.maxY - this.bound.minY) || 0;

                Css.smartCss(this.$el.firstElementChild, {x: this.value.x, y: this.value.y})
            },
            _getTouchPos(e) {
                var tp = e.changedTouches ? e.changedTouches[0] : e;
                return {pageX: tp.pageX, pageY: tp.pageY};
            },
            _onTouchStart(e) {
                if (this.onlyExternal) {
                    return;
                }
                Css.smartCss(this.$el.firstElementChild, {transition:'all 0s ease'});
                this.startPos = this._getTouchPos(e);
                this.startValue = _.extend({}, this.value);
                this.dragging = true;
                document.addEventListener('touchmove', this._onTouchMove, false);
                document.addEventListener('touchend', this._onTouchEnd, false);
                document.addEventListener('mousemove', this._onTouchMove, false);
                document.addEventListener('mouseup', this._onTouchEnd, false);
                this.$emit('start', this.value);
            },
            _onTouchMove(e) {
                var npos = this._getTouchPos(e);
                this.value.x = (npos.pageX - this.startPos.pageX) / this.scale + this.startValue.x;
                this.value.y = (npos.pageY - this.startPos.pageY) / this.scale + this.startValue.y;
                this._matchToBound(this.value);

                this.$emit('update', this.value);
            },
            _onTouchEnd(e) {
                Css.smartCss(this.$el.firstElementChild, {transition:'all '+this.time+'s ease'});
                this.dragging = false;
                document.removeEventListener('touchmove', this._onTouchMove);
                document.removeEventListener('touchend', this._onTouchEnd);
                document.removeEventListener('mousemove', this._onTouchMove);
                document.removeEventListener('mouseup', this._onTouchEnd);
                this.$emit('end', this.value);
            }
        },
        computed: {},
        ready: function () {
            if (!_.isNumber(this.bound.minX)) {
                this.bound.minX = 0;
            }
            if (!_.isNumber(this.bound.maxX)) {
                this.bound.maxX = 100;
            }
            if (!_.isNumber(this.bound.minY)) {
                this.bound.minY = 0;
            }
            if (!_.isNumber(this.bound.maxY)) {
                this.bound.maxY = 0;
            }
            Css.smartCss(this.$el.firstElementChild, {transition:'all '+this.time+'s ease'});
            this._matchToBound(this.value);
        }


    }


</script>