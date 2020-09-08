<template>
<div class="p-8 rounded shadow-2xl md:w-1/2 lg:w-4/6 xl:w-2/6 m-3 lg:m-0" style="background-color: #fefefe;" id="app">
    <div class="flex justify-center">
        <div class="w-2/3 -mt-32 mb-5 p-3 h-48 flex justify-center items-center relative">
            <img class="z-0 absolute pointer-events-none" src="https://cis.visa.com/dam/VCOM/regional/cemea/ukraine/global-elements/icon-logos/visa_cards_VISA_horiz_infinite.png" alt="" srcset="">
            <div class="z-10 w-full flex flex-row space-x-6 justify-center text-white mt-6 text-lg"><span>{{ cardNumber.slice(0, 4) }}</span><span>{{ cardNumber.slice(4, 8) }}</span><span>{{ cardNumber.slice(8, 12) }}</span><span>{{ cardNumber.slice(12, 16) }}</span></div>
            <div class="z-10 text-white absolute" style="top: 75%; left: 15%;">{{ cardName }}</div>
            <div class="z-10 text-white absolute" style="top: 75%; left: 75%;">{{ expirationMonth < 10 && expirationMonth != 0 ? '0' + expirationMonth : expirationMonth }}/{{ expirationYear }}</div>
        </div>
    </div>

    <label class="text-xs font-bold" for="cardNumber" style="color: #7c818c;">Card Number</label>
    <input class="w-full mb-2 border p-2 rounded h-10" type="text" id="cardNumber" v-model="cardNumber" maxlength="16" :class="{'border border-green-300 text-green-300 outline-none':cardNumber.length == 16 && !errors.cardNumber}">
    <span class="text-xs block -mt-2 text-red-500" v-if="errors.cardNumber">{{ errors.cardNumber }}</span>

    <label class="text-xs font-bold" for="cardName" style="color: #7c818c;">Card Name</label>
    <input class="w-full mb-2 border p-2 rounded h-10" type="text" id="cardName" v-model="cardName" maxlength="24" @blur="cardName[--cardName.length] == ' ' ? cardName = cardName.slice(0, -1) : ''" :class="{'border border-green-300 text-green-300 outline-none':cardName.length > 5 && !errors.cardName}">
    <span class="text-xs block -mt-2 text-red-500" v-if="errors.cardName">{{ errors.cardName }}</span>

    <div class="flex flex-row mb-5 mt-1">
        <div class="w-10/12">
            <label class="text-xs font-bold block" for="expirationDate" style="color: #7c818c;">Expiration Date</label>
            <select class="w-5/12 mb-2 border p-2 mr-2 rounded h-10" id="expirationDate" v-model="expirationMonth" :class="{'border border-green-300 text-green-300 outline-none':expirationMonth != '00' && !errors.expirationMonth}">
                <option value="00" disabled>Month</option>
                <option v-if="months" v-for="(key, value) in months" :value="value" :disabled="expirationYear == new Date().getFullYear().toString().slice(2) && --value <= new Date().getMonth()">{{ key }}</option>
            </select>

            <select class="w-5/12 mb-2 border p-2 rounded h-10" id="expirationDate" v-model="expirationYear" :class="{'border border-green-300 text-green-300 outline-none':expirationYear != '00' && !errors.expirationYear}">
                <option value="00" disabled>Year</option>
                <option v-for="year in years" :value="year.short">{{ year.full }}</option>
            </select>

            <span class="text-xs block -mt-2 text-red-500" v-if="errors.expirationDate">{{ errors.expirationDate }}</span>
        </div>

        <div class="w-3/12 -ml-6">
            <label class="text-xs font-bold block" for="CVV" style="color: #7c818c;">CVV</label>
            <input class="w-full border mb-2 p-2 rounded h-10" type="text" id="CVV" maxlength="3" v-model="CVV" :class="{'border border-green-300 text-green-300 outline-none':CVV.length == 3 && !errors.CVV}">
            <span class="text-xs block -mt-2 text-red-500" v-if="errors.CVV">{{ errors.CVV }}</span>
        </div>
    </div>

    <button class="block w-full p-2 rounded shadow-md text-white focus:shadow-outline" style="background-color: #0055d4;" @click="sendData">Submit</button>

    <div v-if="status > 0" class="fixed top-0 left-0 w-full h-full flex items-center justify-center cursor-wait z-50" style="background-color: rgba(240, 240, 240, 0.9)">
        <svg v-if="status == 1" width="25" height="25" viewBox="0 0 50 50" style="enable-background:new 0 0 50 50;">
            <path fill="#000" d="M43.935,25.145c0-10.318-8.364-18.683-18.683-18.683c-10.318,0-18.683,8.365-18.683,18.683h4.068c0-8.071,6.543-14.615,14.615-14.615c8.072,0,14.615,6.543,14.615,14.615H43.935z">
                <animateTransform attributeType="xml" attributeName="transform" type="rotate" from="0 25 25" to="360 25 25" dur="0.6s" repeatCount="indefinite"></animateTransform>
            </path>
        </svg>

        <div v-if="status == 2" class="w-1/2 md:w-2/6 lg:w-1/6 p-5 bg-white rounded shadow-xl text-green-500 text-center cursor-default">Success!
            <button class="mt-2 block w-full p-1 rounded shadow-md text-white focus:shadow-outline bg-green-400" @click="clearData">Submit</button>
        </div>
    </div>
</div>
</template>

<script>
export default {
    name: 'app',
    data() {
        return {
            status: 0,

            cardNumber: localStorage.getItem('cardNumber') || '',
            cardName: localStorage.getItem('cardName') || '',
            expirationMonth: localStorage.getItem('expirationMonth') || '00',
            expirationYear: localStorage.getItem('expirationYear') || '00',
            CVV: localStorage.getItem('CVV') || '',

            years: [],
            months: {
                1: 'January',
                2: 'February',
                3: 'March',
                4: 'April',
                5: 'May',
                6: 'June',
                7: 'July',
                8: 'August',
                9: 'September',
                10: 'October',
                11: 'November',
                12: 'December'
            },

            errors: {}
        }
    },
    watch: {
        cardNumber: {
            handler: function (val) {
                this.cardNumber = val.replace(/[^0-9]/, '')

                if (val.length > 0) {
                    delete this.errors.cardNumber
                    localStorage.setItem('cardNumber', val)
                } else
                    localStorage.removeItem('cardNumber')
            },
            immediate: true
        },
        cardName: {
            handler: function (val) {
                this.cardName = val.replace(/[^A-z\s]/, '').replace(/\s{2,}/, ' ').replace(/^\s/, '').replace(/(.*)\s(.*)\s/, '$1 $2').replace(/\b\w/g, l => l.toUpperCase())

                if (val.length > 0) {
                    delete this.errors.cardName
                    localStorage.setItem('cardName', val)
                } else
                    localStorage.removeItem('cardName')
            },
            immediate: true
        },
        CVV: {
            handler: function (val) {
                this.CVV = val.replace(/[^0-9]/, '')

                if (val.length > 0) {
                    delete this.errors.CVV
                    localStorage.setItem('CVV', val)
                } else
                    localStorage.removeItem('CVV')
            },
            immediate: true
        },
        expirationMonth: function () {
            if (this.expirationMonth != '00')
                localStorage.setItem('expirationMonth', this.expirationMonth)

            delete this.errors.expirationDate
        },
        expirationYear: function () {
            if (this.expirationYear != '00')
                localStorage.setItem('expirationYear', this.expirationYear)

            delete this.errors.expirationDate
        }
    },
    methods: {
        luhnAlgorithm: function (digits) {
            let sum = 0;
            for (let i = 0; i < digits.length; i++) {
                let cardNum = parseInt(digits[i]);
                if ((digits.length - i) % 2 === 0) {
                    cardNum = cardNum * 2;
                    if (cardNum > 9) {
                        cardNum = cardNum - 9;
                    }
                }
                sum += cardNum;
            }
            return sum % 10 === 0;
        },

        checkFields: function () {

            this.status = 1

            if (this.cardNumber.length != 16)
                this.errors.cardNumber = 'Invalid card number'

            if (this.cardNumber.length == 0)
                this.errors.cardNumber = 'Enter your card number'

            if (this.cardNumber.length == 16)
                if (this.luhnAlgorithm(this.cardNumber) == false)
                    this.errors.cardNumber = 'Yhe card number didn`t pass the verification algorithm'

            if (this.cardName.length < 5 || !/[A-Za-z]{2,}\s[A-Za-z]{2,}/.test(this.cardName))
                this.errors.cardName = 'Invalid name'

            if (this.cardName.length == 0)
                this.errors.cardName = 'Enter your card number'

            if (this.expirationMonth == '00' || this.expirationYear == '00') {
                this.errors.expirationDate = 'Choose'

                if (this.expirationMonth == '00') {
                    this.errors.expirationDate += ' month'
                    if (this.expirationYear == '00')
                        this.errors.expirationDate += ' and year'
                } else if (this.expirationYear == '00') {
                    this.errors.expirationDate += ' year'
                    if (this.expirationMonth == '00')
                        this.errors.expirationDate += ' and month'
                }
            }

            if (this.CVV.length < 3)
                this.errors.CVV = 'Invalid CVV'

            if (this.CVV.length == 0)
                this.errors.CVV = 'Enter CVV'

            this.$forceUpdate()
        },

        sendData: function () {
            this.checkFields()

            if (Object.keys(this.errors).length == 0) {
                // axios...
                setTimeout(() => {
                    this.status = 2
                }, 3000);
            } else
                this.status = 0
        },

        clearData: function () {
            this.cardNumber = '',
                this.cardName = '',
                this.expirationMonth = '00',
                this.expirationYear = '00',
                this.CVV = '',
                localStorage.clear()

            this.status = 0
        }
    },
    created: function () {
        let d = new Date().getFullYear()
        for (let i = 0; i < 10; i++) {
            this.years.push({
                full: d,
                short: d.toString().slice(2)
            });
            d++;
        }
    }
}
</script>

<style>
::-webkit-scrollbar-track {
    background-color: #F5F5F5;
}

::-webkit-scrollbar {
    width: .5em;
    background-color: #F5F5F5;
}

::-webkit-scrollbar-thumb {
    background-color: silver;
}

* {
    font-family: 'Exo 2', sans-serif;
}
</style>
