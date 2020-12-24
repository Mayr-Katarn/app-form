<template>
  <div class="block">
    <div class="container">

      <h3>Основная информация:</h3>
      <div class="element">
        <label for="fio">ФИО</label>
        <input id="fio" type="text" v-model="user.fio" @blur="isFioTouched=true" :class="{'error': isFioTouched && (!user.fio || !isFioValid)}">
        <div class="element__info">
          <span v-if="!user.fio && isFioTouched">Необходимо указать ФИО</span>
          <span v-else-if="!isFioValid && isFioTouched">ФИО должно содержать только буквы</span>
        </div>
      </div>

      <div class="element">
        <label for="phone">Контактный телефон</label>
        <input id="phone" type="text" v-model="user.phone" @blur="isPhoneTouched=true" :class="{'error': isPhoneTouched && (!user.phone || !isPhoneValid)}">
        <div class="element__info">
          <span v-if="!user.phone && isPhoneTouched">Необходимо указать номер телефон</span>
          <span v-else-if="!isPhoneValid && isPhoneTouched">Номер телефона должен начинаться с '+' и не содержать букв или символов кроме '( ) -'</span>
        </div>
      </div>

      <div class="element">
        <label for="email">E-mail</label>
        <input id="email" type="text" v-model="user.email" @blur="isEmailTouched=true" :class="{'error': isEmailTouched && (!user.emailfio || !isEmailValid)}">
        <div class="element__info">
          <span v-if="!user.email && isEmailTouched">Необходимо указать E-mail</span>
          <span v-else-if="!isEmailValid && isEmailTouched">E-mail указан не верно</span>
        </div>
      </div>

      <h3>Проверка доступности региона/города:</h3>
      <div class="element">
        <label for="country">Страна</label>
        <select id="country" v-model="user.country">
          <option disabled value="">Выберите вариант</option>
          <option v-for="item of countries" :key="item.id" :value="item">{{ item.name }}</option>
        </select>
      </div>

      <div v-show="areas.length" class="element">
        <label for="area">Регион</label>
        <select id="area" v-model="user.area">
          <option disabled value="">Выберите вариант</option>
          <option v-for="item of areas" :key="item.id" :value="item">{{ item.name }}</option>
        </select>
      </div>

      <div v-show="cities" class="element">
        <label for="city">Город</label>
        <select id="city" v-model="user.city">
          <option disabled value="">Выберите вариант</option>
          <option v-for="item of user.area.areas" :key="item.id" :value="item">{{ item.name }}</option>
        </select>
      </div>

      <div v-show='validate' class="element">
        <div class="element__info">
          <h4 :class="{'denied': !access}">{{ accessMsg }}</h4>
        </div>
      </div>

      <div class="element">
        <button v-show='validate'>{{ accessBtnMsg }}</button>
      </div>

    </div>
  </div>
</template>

<script>
const regExpEmail = /^[-a-z0-9!#$%&'*+/=?^_`{|}~]+(?:\.[-a-z0-9!#$%&'*+/=?^_`{|}~]+)*@(?:[a-z0-9]([-a-z0-9]{0,61}[a-z0-9])?\.)*(?:aero|arpa|asia|biz|cat|com|coop|edu|gov|info|int|jobs|mil|mobi|museum|name|net|org|pro|tel|travel|[a-z][a-z])$/
const regExpPhone = /^([+][0-9\s-()]{3,25})/i
const regExpFio = /^[a-zA-Zа-яёА-Я-' ]+[a-zA-Zа-яёА-ЯЁ']?$/u

export default {
  data() {
    return {
      user: {
        fio: '',
        phone: '',
        email: '',
        country: '',
        area: '',
        city: ''
      },
      countries: [],
      areas: [],
      cities: false,
      isCountryValid: false,
      isFioTouched: false,
      isPhoneTouched: false,
      isEmailTouched: false,
      access: null,
      accessMsg: '',
      accessBtnMsg: ''
    }
  },
  watch: {
    // Сбрасывает данные и загружает список регионов относительно указанной страны.
    // Если регионов нет - запускает метод fetchAccess.
    'user.country': function() {
      this.user.area = ''
      this.user.city = ''
      this.isCountryValid = false
      this.cities = false
      if (this.user.country) {
        fetch(this.user.country.url)
        .then((response) => {
          return response.json()
        })
        .then((data) => {
          this.areas = data.areas
          if (!data.areas.length) {
            this.fetchAccess()
          }
        })
      }
    },
    // Проверяет наличие городов в укзанном регионе.
    // Если городов нет - запускает метод fetchAccess.
    'user.area': function() {
      this.user.city = ''
      if (this.user.area.areas.length) {
        this.cities = true
        this.isCountryValid = false
      } else {
        this.cities = false
        this.fetchAccess()
      }
    },
    // Проверка верно ли указан город.
    'user.city': function() {
      if (typeof(this.user.city) === 'object') {
        this.fetchAccess()
      } else {
        this.isCountryValid = false
      }
    }
  },
  computed: {
    isFioValid() {
      return regExpFio.test(this.user.fio)
    },
    isPhoneValid() {
      return regExpPhone.test(this.user.phone)
    },
    isEmailValid() {
      return regExpEmail.test(this.user.email)
    },
    validate() {
      return this.isFioValid && this.isPhoneValid && this.isEmailValid && this.isCountryValid
    }
  },
  methods: {
    // Проверка на доступность в городе в виде заглушки и установка ключа валидации указанного местоположения на верное.
    // Доступность устанавливается случайным образом.
    fetchAccess() {
      setTimeout(() => {
        let access = Math.floor(Math.random() * 2)
        if (access === 0) {
          this.access = false
          this.accessMsg = '"Лига роботов" недоступна в данном городе'
          this.accessBtnMsg = 'Уведомить, если освободится'
        } else {
          this.access = true
          this.accessMsg = 'Вы можете открыть "Лигу роботов" в данном городе!'
          this.accessBtnMsg = 'Отправить заявку'
        }
        this.isCountryValid = true
      }, 500)
    }
  },
  // Получение списка городов из справочника.
  created() {
    fetch('https://api.hh.ru/areas/countries')
    .then((response) => {
      return response.json()
    })
    .then((data) => {
      this.countries = data
    })
  }
}
</script>

<style>
body {
  font-family: Arial;
}
  .block {
    display: flex;
    margin-top: 30px;
    justify-content: center;
  }
  .container {
    display: inherit;
    background-color: rgba(0, 170, 210, 0.4);
    padding: 10px 80px;
    flex-direction: column;
    align-items: center;
  }
  h3 {
    font-size: 22px;
    color: rgb(51, 51, 51);
    margin-bottom: 30px;
  }
  h4 {
    font-size: 18px;
    color: rgb(5, 155, 0);
    margin-bottom: 30px;
  }
  .element {
    display: inherit;
    flex-direction: inherit;
    align-items: inherit;
    margin-bottom: 20px;
  }
  .element__info {
    display: inherit;
    justify-content: center;
    height: 20px;
    width: 580px;
  }
  label {
    font-weight: bold;
    font-size: 16px;
    color: rgb(26, 26, 26);
    margin-bottom: 5px;
  }
  input {
    width: 320px;
    height: 24px;
    background-color: rgb(215, 248, 255);
    border: none;
  }
  span {
    color: rgb(172, 0, 0);
    font-size: 14px;
  }
  select {
    width: 324px;
    height: 26px;
    background-color: rgb(215, 248, 255);
  }
  button {
    font-weight: bold;
    font-size: 16px;
    color: white;
    background-color: rgba(0, 172, 210, 0.3);
    width: 220px;
    height: 44px;
    margin-bottom: 12px;
    margin-top: 30px;
    border: none;
    outline: 3px solid rgba(0, 137, 168, 0.6);
    transition: .25s;
  }
  button:hover {
    background-color: rgba(0, 190, 0, 0.6);
    outline: 3px solid rgba(0, 190, 0, 0.6);
    transition: .25s;
    cursor: pointer;
  }
  .error {
    background-color: rgb(255, 215, 215) !important;
  }
  .denied {
    color: rgb(168, 0, 0) !important;
  }
</style>