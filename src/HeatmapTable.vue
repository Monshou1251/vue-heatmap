<template>
    <div class="heatmap-wrapper horizontal">

        <!-- Дни недели -->
        <div class="weekdays-col" :style="{ gap: computeGap, }">
            <div v-for="(day, index) in weekDays" :key="day" class="weekday-label"
                :style="{ height: cellSize, fontSize: fontSize, color: fontColor }">
                <span v-if="props.visibleWeekDays.includes(index)">{{ day }}</span>
            </div>
        </div>

        <div class="heatmap-container">
            <!-- Подписи месяцев -->
            <div class="month-labels-row">
                <div v-for="label in monthLabels" :key="label" class="month-label"
                    :style="{ fontSize: fontSize, color: fontColor }">
                    {{ label }}
                </div>
            </div>

            <!-- Ячейки -->
            <div class="heatmap-grid horizontal" :style="{ gap: computeGap, }">
                <div v-for="(day, i) in calendar" :key="i" class="heatmap-cell" :title="day.date" :style="{
                    backgroundColor: getColor(day.value),
                    width: cellSize,
                    height: cellSize,
                    borderRadius: props.cellRadius,

                }" :class="{ empty: !day.date }" />
            </div>
        </div>
    </div>
</template>




<script setup lang="ts">
import { computed } from 'vue'

interface Day {
    date: string
    value: number
}

interface Props {
    year?: number
    cellSize?: string
    cellRadius?: string
    cellGap?: string
    emptyColor?: string
    colorPalette?: string[]
    data?: Record<string, number>
    visibleWeekDays?: number[]
    fontSize?: string
    fontColor?: string
    locale?: string
}

const props = withDefaults(defineProps<Props>(), {
    year: new Date().getFullYear(),
    cellSize: '14px',
    cellRadius: '3px',
    cellGap: '20%',
    emptyColor: '#151b23',
    colorPalette: () => [
        '#151b23',
        '#033a16',
        '#196c2e',
        '#2ea043',
        '#56d364',
    ],
    data: () => ({}),
    visibleWeekDays: () => [1, 3, 5],
    fontSize: '14px',
    fontColor: "#666",
    locale: 'en',
})

const weekDays = computed(() => {
    const formatter = new Intl.DateTimeFormat(props.locale, { weekday: 'short' })
    const base = new Date(2025, 0, 5) // Воскресенье
    return Array.from({ length: 7 }, (_, i) => {
        const date = new Date(base)
        date.setDate(base.getDate() + i)
        return formatter.format(date)
    })
})

const monthLabels = computed(() => {
    const seen = new Set<string>()
    const formatter = new Intl.DateTimeFormat(props.locale, { month: 'short' })

    return calendar.value
        .filter(day => day.date)
        .map(day => formatter.format(new Date(day.date)))
        .filter(month => {
            if (seen.has(month)) return false
            seen.add(month)
            return true
        })
})


const computeGap = computed(() => {
    if (props.cellGap.includes('%')) {
        const numeric = parseFloat(props.cellGap)
        const size = parseFloat(props.cellSize)
        const unit = props.cellSize.replace(/[0-9.]/g, '') || 'px'
        return `${(numeric / 100) * size}${unit}`
    }

    return props.cellGap
})



function getDaysOfYear(year: number) {
    const days: Day[] = []
    const date = new Date(year, 0, 1)

    while (date.getFullYear() === year) {
        const dateStr = date.toLocaleDateString('sv-SE')
        days.push({ date: dateStr, value: props.data[dateStr] || 0 })
        date.setDate(date.getDate() + 1)
    }

    return days
}

const calendar = computed(() => {
    const days = getDaysOfYear(props.year)

    const firstDate = days[0].date
    const firstDay = new Date(firstDate).getDay()

    const emptyStart = Array.from({ length: firstDay }, () => ({ date: '', value: 0 }))

    const full = [...emptyStart, ...days]

    const pad = (7 - (full.length % 7)) % 7

    const emptyEnd = Array.from({ length: pad }, () => ({ date: '', value: 0 }))

    return [...full, ...emptyEnd]
})

const maxValue = computed(() => {
    const values = Object.values(props.data)
    return values.length ? Math.max(...values) : 1
})

const getColor = (value: number) => {
    if (value === 0) return props.emptyColor

    const palette = props.colorPalette
    const max = maxValue.value

    const level = Math.ceil((value / max) * (palette.length - 1))
    const index = Math.min(level, palette.length - 1)

    return palette[index]
}

</script>

<style scoped>
.heatmap-wrapper {
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    position: relative;
    font-family: system-ui, sans-serif;
    padding-top: 20px;
}


.heatmap-container {
    position: relative;
}

.heatmap-grid {
    display: grid;
}

.horizontal {
    grid-template-rows: repeat(7, auto);
    grid-auto-flow: column;
}

.heatmap-cell {
    box-sizing: border-box;
}

.empty {
    opacity: 0;
}

.weekdays-col {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    margin-right: 6px;
}


.weekday-label {
    font-size: 14px;
    /* text-align: left; */
    color: #666;
    line-height: 1;
    padding: 0 4px;
    display: flex;
    line-height: 1;
    align-items: center;
}

.month-labels-row {
    display: flex;
    justify-content: space-around;
    color: #666;
    position: absolute;
    bottom: 105%;
    left: 0;
    right: 0;
}

.month-label {
    white-space: nowrap;
}
</style>
