<template lang='pug'>
  b-container(fluid)
    b-row(align-h='center')
      b-col(lg='8')
        h1 Whatsapp Рассылка
        b-checkbox(switch v-model='broadcast_mode') Групповая рассылка
        b-alert(:show='broadcast_mode && is_mobile' variant='warning') Возможно вы за мобильным аппаратом, для удобной групповой рекомендуем пользоваться компьютером где включен Web Whatsapp
        b-row
          b-col(md=6)
            div(v-if='broadcast_mode')
              b-textarea(placeholder='Скопируйте сюда ваши номера' v-model='raw_nums' rows=5 style='margin:0.25em 0')
              b-textarea(placeholder='Сообщение (опционально)' v-model='message' style='margin:0.25em 0')
              b-row(v-if='formatted_nums' align-v='center')
                template(v-for='n in formatted_nums')
                  b-col
                    span {{n}}
                  b-col(cols='1')
                    b-checkbox(disabled v-model='num_sent[n]')
                  b-col(cols='5' align='right')
                    b-button(variant='success' @click='openWAlink(n)') Отправить
                  div(class='w-100' style='height:0.5em')
            div(v-else)
              b-input(v-model='raw_num' placeholder='Номер' style='margin:0.25em 0')
              b-textarea(placeholder='Сообщение (опционально)' v-model='message' style='margin:0.25em 0')
              b-row
                b-col
                  p Отправить сообщение на номер: {{formatted_num}}
                  b-button(v-if='formatted_num' :disabled='formatted_num.length<=10' variant='success' @click='openWAlink(formatted_num)') Написать
          b-col(class='d-none d-md-block')
            b-card(v-if='broadcast_mode' title='Рассылка' style='margin:0.25em 0')
              b-card-text()
                p Скопируйте ваши номера разделенные через запятую, или новой строкой: 
                b-card
                  span 87010000001, 8(701)0000002
                  br
                  span +7 (777) 000-00-03
                  br
                  span +77770000004
                p И отправляйте по очереди каждому номеру
                
</template>

<script>

import { isMobile } from 'mobile-device-detect'

export default {
  name: 'App',
  data() {
    return {
      broadcast_mode: false,
      is_mobile: isMobile,
      message: '',
      raw_num: '',
      raw_nums: '',
      num_sent: {}
    }
  },
  methods: {
    formatNumber: function(num){
      let cleaned = num.replace(/\D/g, '')
      let phone_num = cleaned
      let country_code = ''
      if(cleaned.length > 10) {
        country_code = cleaned.substring(0, cleaned.length-10)
        phone_num = cleaned.substring(cleaned.length-10, cleaned.length)
      }
      if(country_code == '8') country_code = '7'
      return country_code + phone_num
    },
    openWAlink: function(formatted_num){
      let url = ''
      if(this.is_mobile)
        url = 'https://api.whatsapp.com/send?phone='+formatted_num+'&text='+encodeURI(this.message)+'&source=&data='
      else
        url = 'https://web.whatsapp.com/send?phone='+formatted_num+'&text='+encodeURI(this.message)+'&source=&data='
      if(formatted_num.length > 10){
        window.open(url)
        if(this.broadcast_mode) this.$set(this.num_sent, formatted_num, true)
      }
    }
  },
  computed: {
    formatted_num: function(){
      return this.formatNumber(this.raw_num)
    },
    formatted_nums: function() {
      if(this.raw_nums.trim()){
        let splitted = this.raw_nums.trim().split(/[\n,]+/)
        return splitted.map(rn => this.formatNumber(rn))
      }
      return []
    }
  }
}
</script>

<style>
</style>
