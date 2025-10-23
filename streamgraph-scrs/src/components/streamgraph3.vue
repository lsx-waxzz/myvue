<template>
    <div class="streamgraph-container">
        <div v-if="data.length > 0">
        <div class="chart-container">
            <div 
            class="chart-info" 
            v-if="currentInfo.name"
            >
            <div class="info">{{ currentInfo.name }}：{{ currentInfo.value}}</div>
            </div>
          <svg :width="width" :height="height" @mousemove="handleMouseMove" @mouseleave="handleMouseLeave">
            <g class="x-axis">
              <line 
            class="axis-line" 
            :x1="margin.left" 
            :y1="height - margin.bottom" 
            :x2="width - margin.right" 
            :y2="height - margin.bottom"
              ></line>
              <text 
            class="axis-label" 
            :x="width/2" 
            :y="height - 5" 
            text-anchor="middle"
              >
            年份
              </text>
            </g>
            <g class="y-axis">
                <line 
            class="axis-line" 
            :x1="margin.left" 
            :y1="margin.top" 
            :x2="margin.left" 
            :y2="height - margin.bottom"
                ></line>
                <text 
            class="axis-label" 
            :x="15" 
            :y="height/2" text-anchor="middle" 
            transform="rotate(-90, 15, 250)"
                >
                名字出现频率
                </text>
                <text 
            class="y-axis-labels" 
            :x="margin.left - 5" 
            :y="margin.top" 
            text-anchor="end"
                >
            {{ maxYvalue }}
                </text>
                <text 
            class="y-axis-labels" 
            :x="margin.left - 5" 
            :y="(height - margin.bottom) / 4" 
            text-anchor="end"
                >
            {{ maxYvalue * 3 / 4 }}
                </text>
                <text 
            class="y-axis-labels" 
            :x="margin.left - 5" 
            :y="height - margin.bottom" 
            text-anchor="end"
                >
                0
                </text>
                <text 
            class="y-axis-labels" 
            :x="margin.left - 5" 
            :y="(height - margin.bottom ) / 2" 
            text-anchor="end"
                >
            {{ (maxYvalue / 2) }}
                </text>
                <text 
            class="y-axis-labels" 
            :x="margin.left - 5" 
            :y="(height - margin.bottom ) * 3 / 4" 
            text-anchor="end"
                >
            {{ maxYvalue / 4}}
                </text>
            </g>
            <g v-for="tick in yearTicks" :key="tick">
                <line 
            :x1="getXPosition(tick)" 
            :y1="height - margin.bottom - 5" 
            :x2="getXPosition(tick)" 
            :y2="height - margin.bottom" 
            stroke="#333" 
            stroke-width="1"
                />
                <text 
            :x="getXPosition(tick)" 
            :y="height - margin.bottom + 18" 
            text-anchor="middle" 
            class="axis-label"
                >
            {{ tick }}
                </text>
            </g>
            <g v-for="(name, index) in names" :key="name">
                <path 
            class="streamgraph-path" 
            :d="generatepath(name)" 
            :fill="colors[index]" 
            :stroke="colors[index]"
            :stroke-width="0.5"
            :opacity="activename === null || activename === name ? 0.85 : 0.15"
            @mouseover="setactivename(name)"
            @mouseout="clearactivename"
            />
            </g>
          </svg>
        </div>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        data: {
            type: Array,
            required: true
        },
        maxYvalue: {
            type: Number,
            required: true,
            default: 1
        }
    },
    data() {
        return {
            width: 1000,
            height: 600,
            margin: {
                top: 20,
                right: 20,
                bottom: 30,
                left: 50
            },
            activename: null,
            colors: [
                '#4e79a7', '#f28e2c', '#e15759', '#76b7b2', '#59a14f',
                '#edc949', '#af7aa1', '#ff9da7', '#9c755f', '#bab0ab',
                '#ff6b6b', '#51cf66', '#ffd43b', '#339af0', '#cc5de8'
            ],
            currentInfo: {
                name: '',
                value: 0
            }
        }
    },
    computed: {
        names() {
            if (this.data.length === 0) return [];
            return Object.keys(this.data[0]).filter(key => key !== 'year');
        },
        yearTicks() {
            if (this.data.length === 0) return [];
            const years = this.data.map(d => d.year);
            const minYear = Math.min(...years);
            const maxYear = Math.max(...years);

            const startYear = Math.floor(minYear / 10) * 10;
            const endYear = Math.ceil(maxYear / 10) * 10;

            const ticks = [];
            for (let year = startYear; year <= endYear; year += 10) {
                ticks.push(year);
            }
            return ticks;
        },
        totalData() {
            if (this.data.length === 0) return [];
            return this.data.map(yearData => {
                let total = 0;
                this.names.forEach(name => {
                    total += yearData[name];
                });
                return { year: yearData.year, total }
            });
        },
        stackedData() {
            if (this.data.length === 0) return [];
            return this.data.map((yearData , index) => {
                const stacked = { year: yearData.year };
                const total = this.totalData[index].total;
                let accumulate = 0;
                this.names.forEach(name => {
                    const value = yearData[name] / total;
                    stacked[`${name}_bottom`] = accumulate;
                    accumulate += value;
                    stacked[`${name}_top`] = accumulate;
                });
                return stacked;
            });
        }
    },
    methods: {
        getXPosition(year) {
            if (this.data.length === 0) return 0;
            const chartWidth = this.width - this.margin.left - this.margin.right;
            const minYear = this.data[0].year;
            const maxYear = this.data[this.data.length - 1].year;
            return this.margin.left + ((year - minYear) / (maxYear - minYear)) * chartWidth;
        },
        getYposition(value) {
            const chartHeight = this.height - this.margin.top - this.margin.bottom;
            const ratio = value;
            return this.height - this.margin.bottom - (ratio * chartHeight);
        },
        generatepath(name) {
            if (this.stackedData.length === 0) return '';
            const top = [];
            const bottom = [];
            this.stackedData.forEach(yearData => {
                const x = this.getXPosition(yearData.year);
                const topY = this.getYposition(yearData[`${name}_top`]);
                const bottomY = this.getYposition(yearData[`${name}_bottom`]);
                top.push({ x, y: topY, year: yearData.year });
                bottom.push({ x, y: bottomY, year: yearData.year });
            });
            let path = `M ${top[0].x} ${top[0].y}`;
            for (let i = 1; i < top.length; i++) {
                const prev = top[i - 1];
                const curr = top[i];
                const point1x = prev.x + (curr.x - prev.x) / 2;
                const point1y = prev.y;
                const point2x = prev.x + (curr.x - prev.x) / 2;
                const point2y = curr.y;
                path += ` C ${point1x} ${point1y} ${point2x} ${point2y} ${curr.x} ${curr.y}`;
            }
            for (let i = bottom.length - 1; i >= 0; i--) {
                const point = bottom[i];
                if (i === bottom.length - 1) {
                    path += ` L ${point.x} ${point.y}`;
                } else {
                    const prev = bottom[i + 1];
                    const point1x = prev.x + (point.x - prev.x) / 2;
                    const point1y = prev.y;
                    const point2x = prev.x + (point.x - prev.x) / 2;
                    const point2y = point.y;
                    path += ` C ${point1x} ${point1y} ${point2x} ${point2y} ${point.x} ${point.y}`;
                }
            }
            path += ` Z`;
            return path;
        },
        setactivename(name) {
            this.activename = name;
        },
        clearactivename() {
            this.activename = null;
            this.currentInfo = { name: '', value: 0 };
        },
        handleMouseMove(event) {
            const svgRect = event.currentTarget.getBoundingClientRect();
            const mouseX = event.clientX - svgRect.left;

            const chartWidth = this.width - this.margin.left - this.margin.right;
            const minYear = this.data[0].year;
            const maxYear = this.data[this.data.length - 1].year;
            const year = Math.round(minYear + ((mouseX - this.margin.left) / chartWidth) * (maxYear - minYear));

            const yearData = this.data.find(d => d.year === year);
            const currentValue = yearData[this.activename];

            this.currentInfo = {
                name: this.activename,
                value: currentValue
            };
        },
        handleMouseLeave() {
            this.clearactivename();
        }
    }
}
</script>

<style scoped>
.axis-label {
    font-size: 13px;
    fill: #555;
    font-weight: 500;
}
.axis-line {
    stroke: #ddd;
    stroke-width: 1;
}
.y-axis-labels {
    font-size: 13px;
    fill: #555;
    font-weight: 500;
}
.chart-container {
    --margin-left: 70px;
    --margin-top: 20px;
    --margin-right: 20px; 
    position: relative;
    overflow: visible;
    padding: 15px;
    min-height: 400px;
    display: flex;
    align-items: center;
    justify-content: center;
}
.chart-info {
    position: absolute;
    left: var(--margin-left); 
    top: var(--margin-top);     
    margin-left: 10px; 
    margin-top: 10px;   
    z-index: 100;  
    padding: 8px 12px;
    font-size: 14px;
}
</style>