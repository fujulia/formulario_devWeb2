<script setup>
import { ref,reactive, computed } from 'vue'
import api from './plugins/axios'
const showProfile = ref(false)
const loadingBar = ref(1)
const loading = ref(true)
const validateData = ref('')
const dataCep = ref([])
const userData = {
  value: '',
  processed: false
}

const user = reactive({
  name: {...userData},
  email: {...userData},
  password: {...userData},
  passwordConfirmation: {...userData},
  dateOfBirth : {...userData},
  cep: {...userData},
  state: {...userData},
  city: {...userData},
  neighborhood: {...userData},
  street: {...userData},
  number: {...userData},
  profilePicture: {...userData},
  hobbies: {
    value:[]
  },
  favoriteLanguage: {...userData},
  biography: {...userData}
})


const states = [
  { uf: 'AC', name: 'Acre' },
  { uf: 'AL', name: 'Alagoas' },
  { uf: 'AP', name: 'Amapá' },
  { uf: 'AM', name: 'Amazonas' },
  { uf: 'BA', name: 'Bahia' },
  { uf: 'CE', name: 'Ceará' },
  { uf: 'DF', name: 'Distrito Federal' },
  { uf: 'ES', name: 'Espírito Santo' },
  { uf: 'GO', name: 'Goiás' },
  { uf: 'MA', name: 'Maranhão' },
  { uf: 'MT', name: 'Mato Grosso' },
  { uf: 'MS', name: 'Mato Grosso do Sul' },
  { uf: 'MG', name: 'Minas Gerais' },
  { uf: 'PA', name: 'Pará' },
  { uf: 'PB', name: 'Paraíba' },
  { uf: 'PR', name: 'Paraná' },
  { uf: 'PE', name: 'Pernambuco' },
  { uf: 'PI', name: 'Piauí' },
  { uf: 'RJ', name: 'Rio de Janeiro' },
  { uf: 'RN', name: 'Rio Grande do Norte' },
  { uf: 'RS', name: 'Rio Grande do Sul' },
  { uf: 'RO', name: 'Rondônia' },
  { uf: 'RR', name: 'Roraima' },
  { uf: 'SC', name: 'Santa Catarina' },
  { uf: 'SP', name: 'São Paulo' },
  { uf: 'SE', name: 'Sergipe' },
  { uf: 'TO', name: 'Tocantins' }
]

const validatePassword = computed(() => {
  return user.password.value == user.passwordConfirmation.value?'is-valid':'is-invalid' }
)

const strongPassword = computed(() => {
  return user.password.value.match(/^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])(?=.*[$*&@#])[0-9a-zA-Z$*&@#]{8,}$/)?'is-valid':'is-invalid'
})

function sendForm() {
  showProfile.value = true
  changeLoadingState()
}

function UploadImage(e) {
  const target = e.target
  if (target && target.files) {
    const file = target.files[0]
    user.profilePicture.value = URL.createObjectURL(file)
  }
}
// animação
function changeLoadingState() {
  setTimeout(() => {
    loading.value = false
  }, 3000)
}
// barra de progresso
function progress(attr) {  
  if (!user[attr].processed & user[attr].value!='') {
    console.log('processar')
    loadingBar.value += (100/13)
    user[attr].processed = true
  } else {
    console.log("Nao processar")
  }
}
// buscar cep
async function searchCep() {
  try {
    const response = await api.get(`/ws/${user.cep.value}/json/`)
    dataCep.value = response.data
    user.city.value = dataCep.value.localidade
    user.street.value = dataCep.value.logradouro
    user.neighborhood.value = dataCep.value.bairro
    loadingBar.value += 36
    console.log(dataCep.value)
  } catch (error) {
    console.log(error)
  }
}
</script>

<template>
  <section v-if="showProfile">
    <div class="loading" v-if="loading">
      <div class="spinner-grow m-2 colorLilac"></div>
      <div class="spinner-grow m-2 colorPurple"></div>
      <div class="spinner-grow m-2 colorLilac"></div>
    </div>
    <div v-else class="contentInfo">
      <div class="d-flex justify-content-center mt-4">
        <div class="profilePicture"><img v-if="user.profilePicture.value" :src="user.profilePicture.value" /></div>
      </div>
      <h1 class="w-100 mt-3 profileTitle">Perfil Criado!</h1>
      <h2 class="w-100 p-2 text-center fs-3">Bem vindo(a) {{ user.name.value }}</h2>
      <div class="w-75 m-auto info">
        <p v-for="(value, key) in user" :key="key">{{ key }} : {{ value.value }}</p>
      </div>
      <div class="w-75 m-auto">
        <button class="btn btn-outline" @click="showProfile = false">Voltar</button>
      </div>
    </div>
  </section>
  <form v-else @submit="sendForm()" :class="[validateData]" validate>
    <div
      class="progress mt-2 mb-5 sticky-top"
      role="progressbar"
      aria-label="Basic example"
      aria-valuenow="25"
      aria-valuemin="0"
      aria-valuemax="100"
    >
      <div class="progress-bar m-0" :style="{ width: loadingBar + '%' }"></div>
    </div>

    <div class="m-0 title"><h1>Faça Seu Cadastro</h1></div>
    <div class="form-group">
      <label for="name" >Nome completo*</label>
      <input
        id="name"
        type="text"
        class="form-control"
        placeholder="Nome completo"
        v-model="user.name.value"
        @blur="progress('name')"
        required
      />
      <div class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="form-group">
      <label for="email">E-mail*</label>
      <input 
        id="email"
        type="email"
        class="form-control"
        placeholder="E-mail"
        v-model="user.email.value"
        @blur="progress('email')"
        required
      />
      <div class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="form-group">
      <label for="password">Senha*</label>
      <input
        id="password"
        type="password"
        class="form-control" :class="strongPassword"
        placeholder="Senha"
        v-model="user.password.value"
        @blur="progress('password')"
        required
      />
      <div class="invalid-feedback">
        Insira uma senha que inclua letras maiúsculas, minúsculas, caracteres especiais e números.
      </div>
    </div>
    <div class="form-group">
      <label for="passwordConfirmation">Confirme a senha*</label>
      <input
        id="passwordConfirmation"
        type="password"
        class="form-control" :class="validatePassword"
        placeholder="Confirme a senha"
        v-model="user.passwordConfirmation.value"
        @blur="progress('passwordConfirmation')"
        required 
      />
      <div class="invalid-feedback">Senha incorreta, tente novamente</div>
    </div>
    <div class="form-group">
      <label for="dataOfBirth">Data de nascimento*</label>
      <input
        id="dataOfBirth"
        type="date"
        class="form-control"
        v-model="user.dateOfBirth.value"
        @blur="progress('dateOfBirth')"
      />
      <div class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="form-group">
      <label for="cep">CEP*</label>
      <input
        id="cep"
        type="text"
        class="form-control"
        placeholder="00000000"
        v-model="user.cep.value"
        @blur="searchCep()"
        required
        maxlength="8"
        minlength="8"
      />
      <div class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="form-group">
      <label for="state">Estado*</label>
      <select id="state" name="" class="form-select" required v-model="user.state.value" @blur="progress('state')">
        <option selected disabled value="">Selecionar...</option>
        <option v-for="state of states" :key="state.uf" :value="state.name">
          {{ state.uf }}
        </option>
      </select>
      <div class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="form-group">
      <label for="city">Cidade*</label>
      <input
        id="city"
        type="text"
        class="form-control"
        placeholder="Cidade"
        v-model="user.city.value"
        @blur="progress('city')"
        required
      />
      <div class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="form-group">
      <label for="street">Rua*</label>
      <input
        id="street"
        type="text"
        class="form-control"
        placeholder="Rua"
        v-model="user.street.value"
        @blur="progress('street')"
        required
      />
      <div class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="row">
      <div class="form-group col-8 m-0">
        <label for="neighborhood">Bairro*</label>
        <input
          id="neighborhood"
          type="text"
          class="form-control"
          placeholder="Bairro"
          v-model="user.neighborhood.value"
          @blur="progress('neighborhood')"
          required
        />
        <div class="invalid-feedback">Obrigatório</div>
      </div>
      <div class="form-group col-4 m-0">
        <label for="number">N°*</label>
        <input
          id="number"
          type="text"
          class="form-control"
          placeholder="N°"
          v-model="user.number.value"
          @blur="progress('number')"
          required
        />
        <div class="invalid-feedback">Obrigatório</div>
      </div>
    </div>
    <div class="mb-3">
      <label for="formFile" class="form-label">Foto de perfil - Opcional</label>
      <input class="form-control" type="file" id="formFile" @change="UploadImage($event)" @blur="progress('profilePicture')" />
    </div>
    <div class="form-group">
      <label for="biography">Biografia - Opcional</label>
      <textarea
        name="biography"
        id="biography"
        rows="8"
        class="form-control"
        placeholder="Escreva sobre você"
        v-model="user.biography.value"
        @blur="progress('biography')"
      ></textarea>
    </div>
    <div class="form-group d-flex flex-column">
      <h2>Hobbies - Opcional</h2>
      <div class="m-3">
        <input
          class="form-check-input m-0"
          type="checkbox"
          v-model="user.hobbies.value"
          value="esporte"
          id="esporte"
        />
        <label for="esporte" class="ms-2 checkbox">Esportes</label>
      </div>
      <div class="m-3">
        <input class="form-check-input m-0" type="checkbox" v-model="user.hobbies.value" value="música" id="musica" />
        <label for="musica" class="ms-2 checkbox">Música</label>
      </div>
      <div class="m-3">
        <input
          class="form-check-input m-0"
          type="checkbox"
          v-model="user.hobbies.value"
          value="viagens"
          id="viagens"
        />
        <label for="viagens" class="ms-2 checkbox">Viagens</label>
      </div>
      <div class="m-3">
        <input
          class="form-check-input m-0"
          type="checkbox"
          v-model="user.hobbies.value"
          value="leitura"
          id="leitura"
        />
        <label for="leitura" class="ms-2 checkbox">Leitura</label>
      </div>
      <div class="m-3">
        <input class="form-check-input m-0" type="checkbox" v-model="user.hobbies.value" value="Outro" id="outro" />
        <label for="outro" class="ms-2 checkbox">Outro</label>
      </div>
    </div>
    <div class="form-group">
      <h2>Linguagem preferida - Opcional</h2>
      <div class="form-check m-4">
        <input
          class="form-check-input m-0"
          type="radio"
          name="flexRadioDefault"
          id="flexRadioDefault1"
          v-model="user.favoriteLanguage.value"
          value="javascript"
        />
        <label class="form-check-label ms-2 radio" for="flexRadioDefault1">Javascript</label>
      </div>
      <div class="form-check m-4">
        <input
          class="form-check-input m-0"
          type="radio"
          name="flexRadioDefault"
          id="flexRadioDefault2"
          v-model="user.favoriteLanguage.value"
          value="python"
        />
        <label class="form-check-label ms-2 radio" for="flexRadioDefault2"> Python</label>
      </div>
      <div class="form-check m-4">
        <input
          class="form-check-input m-0"
          type="radio"
          name="flexRadioDefault"
          id="flexRadioDefault3"
          v-model="user.favoriteLanguage.value"
          value="C"
        />
        <label class="form-check-label ms-2 radio" for="flexRadioDefault3">C</label>
      </div>
      <div class="form-check m-4">
        <input
          class="form-check-input m-0"
          type="radio"
          name="flexRadioDefault"
          id="flexRadioDefault4"
          v-model="user.favoriteLanguage.value"
          value="PHP"
        />
        <label class="form-check-label ms-2 radio" for="flexRadioDefault4">PHP</label>
      </div>
      <div class="form-check m-4">
        <input
          class="form-check-input m-0"
          type="radio"
          name="flexRadioDefault"
          id="flexRadioDefault5"
          v-model="user.favoriteLanguage.value"
          value="C++"
        />
        <label class="form-check-label ms-2 radio" for="flexRadioDefault5"> C++</label>
      </div>
      <div class="form-check m-4">
        <input
          class="form-check-input m-0"
          type="radio"
          name="flexRadioDefault"
          id="flexRadioDefault6"
          v-model="user.favoriteLanguage.value"
          value="outra"
        />
        <label class="form-check-label ms-2 radio" for="flexRadioDefault6"> Outra </label>
      </div>
    </div>

    <button type="submit" class="btn btn-outline" @click="validateData = 'was-validated'">
      Cadastrar
    </button>
  </form>
</template>

<style scoped>
form {
  width: 80%;
  margin: auto;
  font-size: 18px;
}
form div {
  margin-top: 5vh;
}

.progress-bar {
  background-color: #5bff76;
}

.title {
  width: 100%;
  display: flex;
  justify-content: center;
  margin-bottom: 4vh;
}

.profileTitle {
  font-size: 30px;
}

h1 {
  width: 5em;
  text-align: center;
  font-size: 36px;
  color: #3300ff;
}

.form-control,
.form-select {
  border: 1px solid #6f6d77;
  padding: 10px;
  margin-top: 5px;
}

input,
select,
.form-control::file-selector-button {
  color: #595c5f;
}

input:focus,
textarea:focus,
select:focus {
  outline: none;
  box-shadow: 0 0 0 0.2rem rgba(125, 95, 245, 0.425);
  border: 1px solid #7d5ff5;
}

label,
h2 {
  color: #7d5ff5;
  font-weight: 300;
  font-size: 18px;
}

.form-check-input:checked {
  background-color: #3300ff;
  border: 1px solid #3300ff;
}

.checkbox,
.radio,
.was-validated .form-check-input:valid ~ .form-check-label {
  color: #595c5f;
}

.form-check {
  padding-left: 0;
}

.was-validated .form-check-input:valid,
.form-check-input {
  border: 1px solid #3300ff;
}

button {
  width: 100%;
  background-color: #3300ff;
  padding: 15px;
  color: white;
  font-size: 20px;
  margin: 2vh 0 3vh 0;
}

button:hover {
  background-color: #7d5ff5;
  color: white;
}

.profilePicture img {
  width: 150px;
  height: 150px;
  border-radius: 50%;
}

section {
  word-wrap: break-word;
}

@keyframes emerge{
  0% {
    opacity: 0;
  }

  100% {
    opacity: 1;
  }
}

.contentInfo {
  animation: emerge 1.8s;
}

.loading {
  width: 100%;
  height: 100%;
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
}
.colorLilac {
  color: #7d5ff5;
}

.colorPurple {
  color: #3300ff;
}
.spinner-grow {
  animation-iteration-count: 4;
}

.was-validated .form-control:valid,
.was-validated .form-select:valid {
  border-color: #6f6d77;
}

.was-validated .form-check-input:valid:checked {
  background-color: #3300ff;
}

.invalid-feedback {
  margin-top: 10px;
}

@media (min-width: 500px) {
  form {
    height: 90vh;
    overflow-y: auto;
    padding: 15px;
    width: 100%;
  }

  .loading {
    width: 500px;
  }
}
</style>
