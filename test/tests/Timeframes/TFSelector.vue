<template>
<!-- Timeframe Selector -->
<div class="tf-selector">
    <span class="timeframe" v-for="(tf, i) in this.timeframes"
        v-on:click="on_click(tf, i)"
        v-bind:style= "selected === i ? {color: '#44c767'} : {}">
        {{tf}}
    </span>
</div>
</template>

<script>
export default {
    name: 'TfSelector',
    props: ['charts'],
    mounted() {
        this.$emit('selected', {
            name: this.timeframes[this.selected],
            i: this.selected
        })
    },
    computed: {
        timeframes() {
            return Object.keys(this.$props.charts)
        }
    },
    methods: {
        on_click(tf, i) {
            this.selected = i
            this.$emit('selected', {
                name: this.timeframes[this.selected],
                i: this.selected
            })
        }
    },
    data() {
        return {
            selected: 0
        }
    }
}
</script>

<style>
.tf-selector {
  position: absolute;
  top: 40px;
  left: 68px;
  font: 14px -apple-system,BlinkMacSystemFont, Segoe UI,Roboto,Oxygen,Ubuntu,Cantarell, Fira Sans,Droid Sans,Helvetica Neue, sans-serif;
  background: #121826e6;
  color: #ccc;
  padding: 8px;
  border-radius: 3px;
  border: 1px solid #2f3240d6;
}
.timeframe {
    margin-right: 5px;
    user-select: none;
    cursor: pointer;
    font-weight: 200;
    max-width: 10px;
}
.timeframe:hover {
    color: #fff;
}
</style>
