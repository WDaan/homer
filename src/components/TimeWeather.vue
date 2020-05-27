<template>
    <article v-if="item" class="message" :class="item.style">
        <div class="columns">
            <div
                v-if="item.time"
                style="margin-top: 1em;"
                class="column has-text-centered"
            >
                <h1 class="is-1">{{ time }}</h1>
            </div>
            <div
                style="margin-top: 1em;"
                class="column has-text-centered"
                v-if="item.weather"
            >
                <div class="columns" style="max-width: 50%; margin: 0 auto;">
                    <div class="column" style="margin-top: 0em;">
                        <span class="is-size-4">
                            {{ weather.temperature }}°
                        </span>
                    </div>
                    <div class="column" style="margin-top: -0.6em;">
                        <span v-html="weather.icons[0]"></span>
                    </div>
                    <div class="is-size-5 column is-uppercase">
                        {{ item.city }}
                    </div>
                </div>
            </div>
        </div>
    </article>
</template>

<script>
import twemoji from 'twemoji'
import moment from 'moment-timezone'
import Weather from '../services/Weather'

function emoji(character) {
    return twemoji.parse(character)
}

export default {
    name: 'TimeWeather',
    props: {
        item: Object
    },
    data: () => ({
        time: '',
        weather: {
            temperature: '',
            icons: []
        },
        rainForecasts: [],
        weatherApi: null,
        weatherInterval: null,
        timeInterval: null
    }),
    created() {
        if (this.item.time)
            this.timeInterval = setInterval(this.getTime(), 1000)
        this.getWeather()
    },
    methods: {
        getTime() {
            this.time = moment().tz(this.item.timezone).format('HH:mm')
        },
        getWeather() {
            if (this.item.weather) {
                this.weatherApi = new Weather(this.item.token)

                this.fetchWeather()
                this.weatherInterval = setInterval(
                    this.fetchWeather,
                    15 * 60 * 1000
                )
            }
        },
        async fetchWeather() {
            const condition = await this.weatherApi.forCity(this.item.city)
            let icons = []
            condition.weather
                .slice(0, 1) // There's not enough room for > 1 emoji -> only display the first weather condition
                .forEach(weatherCondition => {
                    const isNight = weatherCondition.icon.includes('n')
                    const icon = this.weatherApi.getEmoji(
                        weatherCondition.id,
                        isNight
                    )
                    icons.push(emoji(icon))
                })
            this.weather.temperature = condition.main.temp.toFixed(1)
            this.weather.icons = icons
        }
    },
    beforeDestroy() {
        clearInterval(this.timeInterval)
        clearInterval(this.weatherInterval)
    }
}
</script>

<style>
.emoji {
    width: 3em !important;
}
</style>
