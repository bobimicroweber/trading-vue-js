<template>
<!-- Real time data example -->
<span>
    <trading-vue :data="chart" :width="this.width" :height="this.height"
            :chart-config="{MIN_ZOOM:1}"
            ref="tvjs"
            :toolbar="true"
            :index-based="index_based"
            :overlays="overlays"
            :color-back="colors.colorBack"
            :color-grid="colors.colorGrid"
            :color-text="colors.colorText"

           titleTxt="Bitcoin / TetherUS - 1m - BINANCE"

    >
    </trading-vue>
    <span class="gc-mode">
        <input type="checkbox" v-model="index_based">
        <label>Index Based</label>
    </span>

  <tf-selector :charts="charts" v-on:selected="on_selected">
    </tf-selector>

</span>
</template>

<script>
import TradingVue from '../../src/TradingVue.vue'
import TfSelector from './Timeframes/TFSelector.vue'
import Utils from '../../src/stuff/utils.js'
import Const from '../../src/stuff/constants.js'
import DataCube from '../../src/helpers/datacube.js'
import Stream from './DataHelper/stream.js'
import ScriptOverlay from './Scripts/EMAx6.vue'

// Gettin' data through webpeck proxy
const PORT = location.port
const URL = `http://localhost:${PORT}/api/v1/klines?symbol=`
const WSS = `ws://localhost:${PORT}/ws/btcusdt@aggTrade`

export default {
    name: 'SniperCharts',
    icon: '⚡',
    description: 'Real-time updates. Play with DataCube in the console',
    props: ['night'],
    components: {
        TradingVue, TfSelector
    },
    mounted() {
        window.addEventListener('resize', this.onResize)
        this.onResize()

        // Load the last data chunk & init DataCube:
        let now = Utils.now()
        this.load_chunk([now - Const.HOUR4, now], this.waw).then(data => {
            this.chart = new DataCube({
                ohlcv: data['chart.data'],
                onchart: [],
                offchart: [],
                datasets: [{
                    type: 'Trades',
                    id: 'binance-btcusdt',
                    data: []
                }]
            }, { aggregation: 100 })

            // Register onrange callback & And a stream of trades
            this.chart.onrange(this.load_chunk)
            this.$refs.tvjs.resetChart()
            this.stream = new Stream(WSS)
            this.stream.ontrades = this.on_trades
            window.dc = this.chart      // Debug
            window.tv = this.$refs.tvjs // Debug

        })

    },
    methods: {
        onResize(event) {
            this.width = window.innerWidth
            this.height = window.innerHeight - 50
        },
        on_selected(tf) {
          this.waw = tf.name;
        },
        // New data handler. Should return Promise, or
        // use callback: load_chunk(range, tf, callback)

        async load_chunk(range, interval = '1m') {
            let [t1, t2] = range
            let x = 'BTCUSDT'
            let q = `${x}&interval=${interval}&startTime=${t1}&endTime=${t2}`
            let r = await fetch(URL + q).then(r => r.json())
            return this.format(this.parse_binance(r))
        },
        // Parse a specific exchange format
        parse_binance(data) {
            if (!Array.isArray(data)) return []
            return data.map(x => {
                for (var i = 0; i < x.length; i++) {
                    x[i] = parseFloat(x[i])
                }
                return x.slice(0,6)
            })
        },
        format(data) {
            // Each query sets data to a corresponding overlay
            return {
                'chart.data': data
                // other onchart/offchart overlays can be added here,
                // but we are using Script Engine to calculate some:
                // see EMAx6 & BuySellBalance
            }
        },
        on_trades(trade) {
            this.chart.update({
                t: trade.T,     // Exchange time (optional)
                price: parseFloat(trade.p),   // Trade price
                volume: parseFloat(trade.q),  // Trade amount
                'datasets.binance-btcusdt': [ // Update dataset
                    trade.T,
                    trade.m ? 0 : 1,          // Sell or Buy
                    parseFloat(trade.q),
                    parseFloat(trade.p)
                ],
                // ... other onchart/offchart updates
            })
        }
    },
    beforeDestroy() {
        window.removeEventListener('resize', this.onResize)
        if (this.stream) this.stream.off()
    },
    computed: {
        colors() {
            return this.$props.night ? {} : {
                colorBack: '#fff',
                colorGrid: '#eee',
                colorText: '#333'
            }
        },
    },
    data() {
        return {
            chart: {},
            charts: {
              //"1s":"1s",
              "1m":"1m",
              "3m":"3m",
              "5m":"5m",
              "15m":"15m",
              "30m":"30m",
              "1h":"1h",
              "2h":"2h",
              "4h":"4h",
              "6h":"6h",
              "8h":"8h",
              "12h":"12h",
              "1d":"1d",
              "3d":"3d",
              "1w":"1w",
              "1M":"1M",
            },
            width: window.innerWidth,
            height: window.innerHeight,
            index_based: false,
            overlays: [ScriptOverlay]
        }
    }
}
</script>
