<template lang="html">
  <main >
  <section id="section" :class="attr['section']">
     <div :class="attr['section__controller']">
      <div :class="attr['section__title']">
        <a href="#" @click.prevent="prevMonth(active_date)" :class="attr['section__title--small']">Prev</a> 
        {{ getMonth(active_date) }}
        <a href="#" @click.prevent="nextMonth(active_date)" :class="attr['section__title--small']">Next</a>
      </div>

      <div :class="attr['section__todo-container']">
        <span :class="attr['section__label']">Input Todo</span>
        <input :class="attr['section__input-todo']" type="text" name="todo">
      </div>
      
      <div :class="attr['section__date-picker']">
        <div :class="attr['section__left']" >
          <span>Start Date</span>
          <input type="date" name="end" v-model="date1">
        </div>
        <div :class="attr['section__right']" >
          <span>End Date</span>
          <input type="date" name="end" v-model="date2" >
        </div>
      </div>

      <div :class="attr['section__button-container']">
        <button :class="attr['section__button']" > Save</button>
        <button :class="[attr['section__button'], attr['section__button--override']]"> Override</button>
      </div>
    </div>

   <!--  <div :class="attr['section__controller']">
      
    </div> -->
    <div :class="attr['section__calendar']" id="calendar">
    </div>
  </section>

  </main>
</template>

<script>
  import moment from 'moment'
  export default {
    // components: {
    //   PageContent: () => import('~/components/home/PageContent')
    // },
    
    data: () => ({
      loaded: false,
      records: null,
      date1: '',
      date2: '',
      page: {
        title: "Home",
      },
      calendar: [],
      selectedDate: [],
      active_date: '',
      day: [
        'Sun','Mon','Tue','Wed','Thu','Fri','Sat'
      ]
    }),
    updated() {
      this.$nuxt.$emit("toggle-footer", "image", true);
    },
    methods: {
      getDate($date) {
        let currentMonthFormat = this.getMonth(this.active_date),
            dateMonthFormat    = this.getMonth($date),
            currentDayFormat = moment().format('DD-MM').toString(),
            dateDayFormat    = moment($date).format('DD-MM').toString()

        if(currentMonthFormat != dateMonthFormat) {
          return `<td style="background-color: #DFDFDE">${moment($date).format('D').toString()}</td>`
        } else {
          if(currentDayFormat == dateDayFormat) {
            return `<td style="background-color: #CDF0EA">${moment($date).format('D').toString()}</td>`
          } 
          return `<td>${moment($date).format('D').toString()}</td>`
        }
      },
      getMonth($month) {
        let date = moment($month).format('MMMM').toString() + ' ' + moment($month).year()
        return date
      },
      prevMonth($date) {
        let month_count  = moment($date).month() + 1
        const startWeek = moment($date).subtract(1, "month").startOf('month').week()
        const endWeek = moment($date).subtract(1, "month").endOf('month').week() + 1

        if(month_count == 1 ) {
          const date = moment($date).subtract(1, "year")
          const start = moment(date).add(10, "month").startOf('month').week() + 4
          const end = moment(date).add(10, "month").endOf('month').week() + 5
          this.calendarGetter(start, end, date)
        } else {
          this.calendarGetter(startWeek, endWeek, $date)
        }
      },
      nextMonth($date) {
        let month_count  = moment($date).month() + 1
        const startWeek = moment($date).add(1, "month").startOf('month').week()
        const endWeek = moment($date).add(1, "month").endOf('month').week() + 1
        if(month_count == 12) {
          const date = moment($date).add(1, "year")
          const start = 1
          const end = 6
          this.calendarGetter(start, end, date )
        } else if (month_count == 11) {
          const startWeek = moment($date).add(1, "month").startOf('month').week()
          const endWeek = moment($date).add(1, "month").endOf('month').week()
          this.calendarGetter(startWeek, endWeek, $date )
        } else {
          this.calendarGetter(startWeek, endWeek, $date)
        }
      },
      calendarDateSetter() {
        const startWeek = moment().startOf('month').week()
        const endWeek = moment().endOf('month').week() + 1
        this.calendarGetter(startWeek, endWeek)
      },
      calendarGetter($startWeek, $endWeek, $date = moment(), $set) {
        let calendar = [],
            date     = $date

        for(var week = $startWeek; week<$endWeek;week++){
          calendar.push({
            week:week,
            days:Array(7).fill(0).map((n, i) => moment($date).week(week).startOf('week').clone().add(n + i, 'day'))
          })
        }
        this.calendar = calendar
        if(this.calendar){
          this.active_date = calendar[1].days[0]
          this.calendarSet()
        }
      },
      calendarSet () {
        let element = document.querySelector("#calendar"),
            page = "<table>"
            page += `<thead><tr>`
            this.day.forEach((value, index) => {
              page += `<th>${value}</th>`
            })
            page += `</tr></thead>`
            page += "<tbody>"
            this.calendar.forEach((week, index) => {
              page += `<tr>`
                week.days.forEach((day, index) => {
                  page += this.getDate(day._d)
                })
              page += `</tr>`
            })
            page += '</tbody>'
            page += '</table>'
        element.innerHTML = page
      }
    },
    mounted() {
      setTimeout(() => {
        this.toggleModalStatus({
          type: "loader",
          status: true,
          item: { start: false },
        });
        this.calendarDateSetter()
        this.loaded = true;
      }, 500);
    
    },
    asyncData({ $axios, store, error }) {
      store.commit("global/modal/toggleModalStatus", {
        type: "loader",
        status: true,
        item: { start: true },
      });

      // return $axios.$get('web/page/landing/home').then(({
      //   records
      // }) => {

      // }).catch(() => {
      //   error({ statusCode: 500 })
      // })
    },
    head() {
      let host = process.env.BASE_URL;

      return {
        title: `Propan | ${this.page.title}`,
        link: [
          {
            rel: "canonical",
            href: `${host}${this.$route.fullPath}`,
          },
        ],
        // meta: [
        //   { hid: 'description', name: 'description', content: `${this.page.metadata.meta_description}` },
        //   { hid: 'og:title', property: 'og:title', content: `${this.page.metadata.meta_title}` },
        //   { hid: 'og:description', property: 'og:description', content: `${this.page.metadata.meta_description}` },
        //   { hid: 'og:url', property: 'og:url', content: `${host}${this.$route.fullPath}` },
        //   { hid: 'og:image', property: 'og:image', content: `${host}/logo.svg` },
        //   { hid: 'og:image:alt', property: 'og:image:alt', content: this.page.title },
        //   { hid: 'og:type', property: 'og:type', content: 'website' },
        //   { hid: 'og:site_name', property: 'og:site_name', content: 'C! Magazine' },
        // ]
      };
    },
  };
</script>

<style lang="stylus" module="attr">
  .section
    display: flex
    justify-content: space-between
    flex-flow: row wrap
    width: 100%
    max-width: 1280px
    margin: 100px auto 0
    padding: 80px 20px 60px 20px
    background-color: #efefef
    border-radius: 10px
    box-shadow: -4px 10px 21px -4px rgba(0,0,0,0.46)
    
    &__controller
      flex: 0 0 calc((40%) - 20px)
      & ^[0]__title
        display: flex
        justify-content: space-between
        align-items: center
        font-size: 30px
        margin-bottom: 30px
        &--small
          font-size: 18px
          color: #000
        &--small:hover
          text-decoration: underline
      & ^[0]__todo-container
        & ^[0]__input-todo
          border: 1px solid #000     
          padding: 0px 10px
          width: 100%     
          height: 40px
          font-size: 18px
        & ^[0]__label
          font-size: 18px
          margin-bottom: 15px
      & ^[0]__date-picker
        display: flex
        flex-flow: row wrap
        justify-content: space-between
        padding-top: 25px
        & ^[0]__left, & ^[0]__right
          flex: 0 0 calc((100% / 2) - 10px)
          input
            width: 100%
            padding-left: 10px
            border: 1px solid #000
            height: 40px
            font-size: 18px
      & ^[0]__button-container
        display: flex
        flex-flow: row wrap
        justify-content: space-between
        margin-top: 30px
        
        & ^[0]__button
          flex: 0 0 calc((100% / 2 ) - 10px)
          width: 100%
          height: 35px
          border: 1px solid #000
          background-color: #FFFAE7
          &--override
            background-color: #CDF0EA
        
        & ^[0]__button:hover
          background-color: #59CE8F
          color: #fff
          box-shadow: 1px 6px 25px -4px rgba(0,0,0,0.39)     
             
    &__calendar
      position: relative
      flex: 0 0 calc(60% - 20px)
      thead, tbody
        border: 1px solid #000
      thead
        tr
          th
            width: 90px
            padding: 10px 0
            fonts-size: 18px
            font-weight: 500
            border: 1px solid #000
      tbody
        tr
          td
            width: 90px
            padding: 15px 10px 45px
            border: 1px solid #000
            font-size: 16px

</style>