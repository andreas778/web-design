Назва лаб. роботи - Розробка Web-додатка засобами Javascript/VueJS, Завдання - адаптувати програмний код ЛР№2 до вимог фреймворка VueJS та забезпечити завантаження необхідних даних з Web-сервера, Посилання на звіт - https://docs.google.com/document/d/1_YDySvW4Fy31BaETNTylJHHm5pl-gb9vS59yLdRzPls/edit

Мета: ознайомитись із засобами фреймворка VueJS та навчитись створювати асинхронні запити до Web-сервера.
Загальне завдання: адаптувати програмний код ЛР№2 до вимог фреймворка VueJS та забезпечити завантаження необхідних даних з Web-сервера.
Інструменти розробки: Javascript, VueJS, NodeJS, Express.



Назва додатку: ЧасоМетр
Короткий опис його функціональності:
Веб-додаток, який реалізує облік робочого часу (запуск таймера, призупинення/продовження, зупинка, збереження назви, часу початку і завершення сеансу роботи).



Схема взаємодії VueJS-компонентів



Скріни усіх сторінок Web-додатка
Головна






Про нас




Зареєструватися






Увійти


Профіль




Код компонентів
App.vue
<template>
  <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
  <div class="container">
            <a class="navbar-brand" href="#">ЧасоМетр</a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav">
                    <li class="nav-item">
                        <router-link to="/" class="nav-link" >Головна</router-link>
                    </li>
                    <li class="nav-item">
                         <router-link to="/about" class="nav-link" >Про нас</router-link>
                    </li>
                    <li class="nav-item">
                        <router-link to="/profile" class="nav-link" >Профіль</router-link>
                    </li>
                    <li class="nav-item">
                        <router-link to="/signin"  class="nav-link" v-if="!store.state.currentUser">Увійти</router-link>
                    </li>
                    <li class="nav-item">
                        <router-link to="/signup" class="nav-link" v-if="!store.state.currentUser">Зареєструватися</router-link>
                    </li>
                    <li class="nav-item">
                        <router-link to=""  class="nav-link" v-if="store.state.currentUser" @click="store.commit('logOut')">Вийти</router-link>
                    </li>
                </ul>
            </div>
        </div>   
  </nav>
  <router-view/>
</template>

<script setup>
import store from './store';
</script>

Main.vue
<template>
  <div class="container mt-5 main-container">
        <div class="row">
            <div class="col">
                <h1>Таймер</h1>
                <div id="timer">{{this.timer.currentTime}}</div>
                <div class="mt-3">
                    <button id="startBtn" class="btn btn-primary" @click="this.startTimer()">Старт</button>
                    <button id="pauseBtn" class="btn btn-secondary" @click="pauseTimer()">Пауза</button>
                    <button id="stopBtn" class="btn btn-danger" @click="stopTimer()">Стоп</button>
                    <button id="saveBtn" class="btn btn-success" @click="saveSession()">Зберегти</button>
                </div>
            </div>
            <div class="col">
                <h1>Інформація про робочий час</h1>
                <p id="workLog" v-html="this.timer.workLog"></p>
                <h1>Історія сеансів</h1>
                <ul id="sessionHistory" v-html="this.timer.sessionHistory"></ul>
            </div>
        </div>
        <footer class="footer bg-primary">
            2024 ЧасоМетр ©
        </footer>
    </div>
</template>

<script>
import api from '@/services/api';
import store from '../store';

export default {
  name: 'Main', 
  data() {
        return {
            timer:{
                startTime: 0,
                startTimeDate: 0,
                currentTime: this.formatTime(0),
                pauseTime: null,
                pauseDurations: [],
                currentSession: {
                  startTime: null,
                  pauseDurations: []
                },
                sessions: [],
                timeoutId: null,
                workLog: ``,
                sessionHistory: ``,
            }
        }
    },
    
    mounted() {
        if (store.state.currentUser !== null) {
            this.timer.sessions = JSON.parse(store.state.currentUser.sessions);
            
            this.timer.sessions.forEach((session) => {
                this.updateSessionHistory(session)
            });
        }
    },
    
    methods: {        
        startTimer() {
            //this.fetchUsers();
            if (this.timer.startTime === 0 || this.timer.pauseTime !== null) {
                if (this.timer.pauseTime !== null) {
                    this.resumeTimer();
                }
                else {
                    this.timer.currentSession.startTime = new Date();
                    this.timer.workLog = `Початок: ${this.formatDateTime(this.timer.currentSession.startTime)}`;
                }
                this.timer.currentTime = this.formatTime(this.timer.startTime);
                this.countTimer ();
            }
        },
      
      countTimer () {
            this.timer.timeoutId = setTimeout(() => {
                this.timer.startTime += 1000;
                this.timer.currentTime = this.formatTime(this.timer.startTime);
                this.countTimer()
            }, 1000)             
      },
        
      pauseTimer() {
          if (this.timer.pauseTime === null && this.timer.startTime > 0) {
            this.timer.pauseTime = new Date();
            clearTimeout(this.timer.timeoutId);    
          }
      },

      resumeTimer() {
        if (!this.timer.pauseDurations.length){
             this.timer.workLog += `<br>Паузи:`
        }
        const pauseDuration = {
            pauseStartTime: this.timer.pauseTime,
            pauseEndTime: new Date()
        };
        this.timer.pauseDurations.push(pauseDuration);
        this.timer.currentSession.pauseDurations.push(pauseDuration);
        this.timer.workLog += `<li>${this.formatDateTime(this.timer.pauseTime)} - ${this.formatDateTime(new Date())}</li>`;
        this.timer.pauseTime = null;
      },
      
      
    saveUserSession() {
        store.state.currentUser.sessions = JSON.stringify(this.timer.sessions);
        const response = api.updateUserSessions(store.state.currentUser.name, store.state.currentUser.sessions);
      },
      
    stopTimer() {
        if (confirm('Чи бажаєте ви зберегти сеанс?')) {        
            if (store.state.currentUser === null) {
                alert('Для ціє дії треба увійти!');
                return;
            }
            const sessionName = prompt('Введіть назву сеансу:');
            if (sessionName) {
                const endTime = new Date();
                const totalDuration = endTime.getTime() - this.timer.currentSession.startTime.getTime();
                const session = {
                    name: sessionName,
                    startTime: this.timer.currentSession.startTime,
                    pauseDurations: this.timer.currentSession.pauseDurations,
                    endTime: endTime,
                    totalDuration: totalDuration
                };
                this.updateSessionHistory(session);
                this.timer.sessions.push(session);
                this.saveUserSession();
            }
        }
        clearTimeout(this.timer.timeoutId);
        this.resetTimer();    
    },
    
        
    saveSession() {
        if (store.state.currentUser === null) {
            alert('Для ціє дії треба увійти!');
            return;
        }
        const sessionName = prompt('Введіть назву сеансу:');
        if (sessionName) {
            const currentTime = new Date();
            const session = {
                name: sessionName,
                startTime: this.timer.currentSession.startTime,
                pauseDurations: this.timer.currentSession.pauseDurations,
                endTime: currentTime,
                totalDuration: currentTime.getTime() - this.timer.currentSession.startTime.getTime()
            };
            this.updateSessionHistory(session);
            this.timer.sessions.push(session);
            this.saveUserSession();
        }
    },

    updateSessionHistory(session) {
        const startTime = new Date(session.startTime);
        const endTime = new Date(session.endTime);
        const totalDuration = new Date(session.totalDuration);
        const pauseDurations = session.pauseDurations;
        if (pauseDurations.length) {
            this.timer.sessionHistory += `<li>${session.name}<br>Початок: ${this.formatDateTime(startTime)}<br>Паузи:<ul>`;
            pauseDurations.forEach(pauseDuration => {
                this.timer.sessionHistory += `<li>${this.formatDateTime(pauseDuration.pauseStartTime)} - ${this.formatDateTime(pauseDuration.pauseEndTime)}</li>`;
            });
        }
        else {
            this.timer.sessionHistory += `<li>${session.name}<br>Початок: ${this.formatDateTime(startTime)}<ul>`;
        }

        this.timer.sessionHistory += `</ul>Завершено: ${this.formatDateTime(endTime)}
        <br>Тривалість: ${this.formatTime(totalDuration)} </li>`;
    },

      resetTimer() {
        this.timer.startTime = 0;
        this.timer.pauseTime = null;
        this.timer.currentTime = this.formatTime(0),
        this.timer.pauseDurations = [];
        this.timer.currentSession = {
          startTime: null,
          pauseDurations: []
        };
      },
      
      formatTime(time) {
        const hours = Math.floor(time / 3600000)
          .toString()
          .padStart(2, '0');
        const minutes = Math.floor((time % 3600000) / 60000)
          .toString()
          .padStart(2, '0');
        const seconds = Math.floor((time % 60000) / 1000)
          .toString()
          .padStart(2, '0');
        return `${hours}:${minutes}:${seconds}`;
      },

      formatDateTime(dateTime) {
        dateTime = new Date(new Date);
        const year = dateTime.getFullYear();
        const month = (dateTime.getMonth() + 1).toString().padStart(2, '0');
        const day = dateTime.getDate().toString().padStart(2, '0');
        const hours = dateTime.getHours().toString().padStart(2, '0');
        const minutes = dateTime.getMinutes().toString().padStart(2, '0');
        const seconds = dateTime.getSeconds().toString().padStart(2, '0');
        return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
      },
    }
}
</script>

About.vue
<template>
    <div class="container mt-5 main-container">
        <div class="about-container">
            <img src="../assets/logo.png" alt="Емблема" class="logo">
            
            <h2 class="app-title">ЧасоМетр</h2>
            
            <p class="app-description">Додаток "ЧасоМетр" дозволяє ефективно вести облік робочого часу та зберігати історію робочих сеансів.</p>
            
            <div class="app-details">
                <p>З цим додатком ви зможете:</p>
                <ul>
                    <li>Запускати таймер для відстеження часу роботи</li>
                    <li>Призупиняти та продовжувати робочі сеанси за потреби</li>
                    <li>Зберігати історію ваших робочих сеансів для подальшого аналізу</li>
                    <li>І багато іншого!</li>
                </ul>
            </div>
        </div>
        <footer class="footer bg-primary">
            2024 ЧасоМетр ©
        </footer>
  </div>
</template>

Profile.vue
<template>
    <div class="container mt-5 profile-container">
        <div class="profile">
            <table class="profile__table table table-primary">
                <tbody>
                  <tr>
                    <th>Логін</th>
                    <td>{{this.user.name}}</td>
                  </tr>
                  <tr>
                    <th>Email</th>
                    <td>{{this.user.email}}</td>
                  </tr>
                  <tr>
                    <th>Телефон</th>
                    <td>{{this.user.phone}}</td>
                  </tr>
                  <tr>
                    <th>Статистика</th>
                    <td v-html="this.user.stat"></td>
                  </tr>
                </tbody>
              </table>
        </div>
        <footer class="footer bg-primary">
            2024 ЧасоМетр ©
        </footer>
    </div>
</template>

<script>
import store from '../store';

export default {
  name: 'Profile',
  
  data() {
    return {
        user: {
            name: '-',
            email: '-',
            phone: '-',
            stat: '-',
        }
    }
  },
  
  mounted() {
        if (store.state.currentUser !== null) {
            this.user.name = store.state.currentUser.name;
            this.user.email = store.state.currentUser.email;
            this.user.phone = store.state.currentUser.phone;
            this.calculateStat();
        }
    },
    
  methods: {
      calculateStat() {
         if (JSON.parse(store.state.currentUser.sessions).length) {
             this.user.stat = `Кількість сеансів: ${JSON.parse(store.state.currentUser.sessions).length}
             <br>Найдовший сеанс: ${this.formatTime(JSON.parse(store.state.currentUser.sessions).reduce((max, current) => {
            return max.totalDuration > current.totalDuration ? max : current;
            }).totalDuration)}
                <br>Найкоротший сеанс: ${this.formatTime(JSON.parse(store.state.currentUser.sessions).reduce((min, current) => {
                return min.totalDuration < current.totalDuration ? min : current;
            }).totalDuration)}`;
            
            const totalDurationSum = JSON.parse(store.state.currentUser.sessions).reduce((sum, current) => {return sum + current.totalDuration;}, 0);
            const averageTotalDuration = totalDurationSum / JSON.parse(store.state.currentUser.sessions).length;
            
            this.user.stat += `<br>Середня тривалість сеансів: ${this.formatTime(averageTotalDuration)}`;
         }
         else {
             this.user.stat = '-';
         }
      },
      
      formatTime(time) {
        const hours = Math.floor(time / 3600000)
          .toString()
          .padStart(2, '0');
        const minutes = Math.floor((time % 3600000) / 60000)
          .toString()
          .padStart(2, '0');
        const seconds = Math.floor((time % 60000) / 1000)
          .toString()
          .padStart(2, '0');
        return `${hours}:${minutes}:${seconds}`;
      }
  }
}
</script>

Signin.vue
<template>
    <div class="container mt-5 main-container">
        <div class="page__signin signin">
            <div class="signin__container container">
              <form class="main-login__form" action="#">
                <div class="mb-3">
                  <label for="InputLogin" class="form-label">Логін</label>
                  <input class="form-control" type="text" v-model="this.signin.nickname" id="InputLogin" placeholder="Введіть логін">
                </div>
                <div class="mb-3">
                  <label for="InputPassword1" class="form-label">Пароль</label>
                  <input type="password" class="form-control" v-model="this.signin.password" id="InputPassword1" placeholder="Введіть пароль">
                </div>
                <button type="submit" class="btn btn-primary" @click="this.logUser()">Увійти</button>
                <button type="reset" class="btn btn-secondary">Скинути</button>
              </form>
            </div>
          </div>
        <footer class="footer bg-primary">
            2024 ЧасоМетр ©
        </footer>
    </div>
</template>

<script>

import { ref } from 'vue';
import api from '@/services/api';
import store from '../store';
import router from '../router';

export default {
  name: 'Signin',
  
  data() {
      return {
          signin:{
            nickname: ref(''),
            password: ref(''),
          }    
      }
  },
  methods: {
      async logUser() {
          const user = {
              name: this.signin.nickname,
              password: this.signin.password, 
         };
        const response = await api.getUserByName(user.name);
        const userData = response.data.data;
        if (!userData) {
            alert('Логін не знайдено!');
            router.push({ path: 'signin' });
        }
        else if (user.name === userData.name && user.password === userData.password) {
          store.state.currentUser = userData;
          router.push({ path: 'profile' })
          alert('Вхід успішний!');
        }
        else if (user.name === userData.name && user.password !== userData.password) {
          alert('Пароль неправильний!');
            router.push({ path: 'signin' });
        }
      }
  }
}
</script>

Signup.vue
<template>
    <div class="container mt-5 main-container">
        <div class="signup__container container">
          <form class="signup__form" action="#">
            <div class="mb-3">
              <label for="InputNickname1" class="form-label">Логін</label>
              <input class="form-control" type="text" v-model="this.signup.nickname" id="InputNickname1"   placeholder="Введіть логін">
            </div>
            <div class="mb-3">
              <label for="InputEmail1" class="form-label" >Email</label>
              <input type="email" class="form-control" id="InputEmail1" v-model="this.signup.email" placeholder="Введіть email" aria-describedby="emailHelp">
            </div>
            <div class="mb-3">
              <label for="InputTel1" class="form-label" >Телефон</label>
              <input type="tel" class="form-control" id="InputTel1" v-model="this.signup.phone" placeholder="Введіть телефон">
            </div>
            <div class="mb-3">
              <label for="InputPassword1" class="form-label" >Пароль</label>
              <input type="password" class="form-control" id="InputPassword1" v-model="this.signup.password" placeholder="Введіть пароль">
            </div>
            <button type="submit" class="btn btn-primary" @click="this.addUser()">Ок</button>
            <button type="reset" class="btn btn-secondary">Скинути</button>
          </form>
        </div>
        <footer class="footer bg-primary">
            2024 ЧасоМетр ©
        </footer>
    </div>
</template>

<script>
import { ref } from 'vue';
import api from '@/services/api';
import router from '../router';

export default {
  name: 'Signup',
  
  data() {
      return {
          signup:{
            nickname: ref(''),
            email: ref(''),
            phone: ref(''),
            password: ref(''),
            sessions: '[]',
          }    
      }
  },
  methods: {
      addUser() {
          const user = {
              name: this.signup.nickname,
              email: this.signup.email, 
              phone: this.signup.phone, 
              password: this.signup.password, 
              sessions: this.signup.sessions
         };
        const res = api.addUser(user);
        alert('Реєстрація успішна!');
        router.push({ path: 'signin' })
      }
  }
}
</script>



