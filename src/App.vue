<script setup>
import { ref,reactive, computed } from 'vue'
import api from './plugins/axios'
const mostrarPerfil = ref(false)
const barraCaregamento = ref(1)
const loading = ref(true)
const validateData = ref('')
const data = ref([])
const userData = {
  value: '',
  processed: false
}

const user = reactive({
  nome: {...userData},
  email: {...userData},
  senha: {...userData},
  senhaConfirmacao: {...userData},
  dataNascimento: {...userData},
  cep: {...userData},
  estado: {...userData},
  cidade: {...userData},
  bairro: {...userData},
  rua: {...userData},
  num: {...userData},
  fotoPerfil: '',
  hobbies: [],
  linguagemPref: '',
  biografia: ''
})


const estados = [
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

const validaSenha = computed(() => {
  return user.senha.value == user.senhaConfirmacao.value

})
console.log(validaSenha)
const senhaForte = computed(() => {
  return user.senha.value.match(/^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])(?=.*[$*&@#])[0-9a-zA-Z$*&@#]{8,}$/)
})

function enviarForm() {
  mostrarPerfil.value = true
  mudar()
}

function mudar() {
  setTimeout(() => {
    loading.value = false
  }, 3000)
}

function progresso(attr) {  
  if (!user[attr].processed & user[attr].value!='') {
    console.log('processar')
    barraCaregamento.value += (100/11)
    user[attr].processed = true
  } else {
    console.log("Nao processar")
  }
}

async function buscarCep() {
  try {
    const response = await api.get(`/ws/${user.cep.value}/json/`)
    data.value = response.data
    user.cidade.value = data.value.localidade
    user.rua.value = data.value.logradouro
    user.bairro.value = data.value.bairro
    barraCaregamento.value += 36
    console.log(data.value)
  } catch (error) {
    console.log(error)
  }
}

function UploadImagem(e) {
  const target = e.target
  if (target && target.files) {
    const file = target.files[0]
    user.fotoPerfil = URL.createObjectURL(file)
  }
}
</script>

<template>
  <section v-if="mostrarPerfil">
    <div class="loading" v-if="loading">
      <div class="spinner-grow m-2 corLilas"></div>
      <div class="spinner-grow m-2" id="corRoxa"></div>
      <div class="spinner-grow m-2 corLilas"></div>
    </div>
    <div v-else class="contentInfo">
      <div class="d-flex justify-content-center mt-4">
        <div id="fotoPerfil"><img v-if="user.fotoPerfil" :src="user.fotoPerfil" /></div>
      </div>
      <h1 class="w-100 mt-3" id="perfilTitulo">Perfil Criado!</h1>
      <h2 class="w-100 p-2 text-center fs-3">Bem vindo(a) {{ user.nome.value }}</h2>
      <div class="w-75 m-auto info">
        <p v-for="(value, key) in user" :key="key">{{ key }} : {{ value.value }}</p>
      </div>
      <div class="w-75 m-auto">
        <button class="btn btn-outline" @click="mostrarPerfil = false">Voltar</button>
      </div>
    </div>
  </section>
  <form v-else @submit="enviarForm()" :class="[validateData]" validate>
    <div
      class="progress mt-2 mb-5 sticky-top"
      role="progressbar"
      aria-label="Basic example"
      aria-valuenow="25"
      aria-valuemin="0"
      aria-valuemax="100"
    >
      <div class="progress-bar m-0" :style="{ width: barraCaregamento + '%' }"></div>
    </div>

    <div id="titulo" class="m-0"><h1>Faça Seu Cadastro</h1></div>
    <div class="form-group">
      <label>Nome completo*</label>
      <input
        id="validationCustom01"
        type="text"
        class="form-control"
        placeholder="Nome completo"
        v-model="user.nome.value"
        @blur="progresso('nome')"
        required
      />
      <div class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="form-group">
      <label>E-mail*</label>
      <input
        id="validationCustom02"
        type="email"
        class="form-control"
        placeholder="E-mail"
        v-model="user.email.value"
        @blur="progresso('email')"
        required
      />
      <div class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="form-group">
      <label>Senha*</label>
      <input
        id="validationCustom03"
        type="password"
        class="form-control"
        placeholder="Senha"
        v-model="user.senha.value"
        @blur="progresso('senha')"
        required
      />
      <div v-if="!senhaForte && user.senha != ''" class="invalid-feedback">
        Insira uma senha que inclua letras maiúsculas, minúsculas, caracteres especiais e números.
      </div>
      <div v-else class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="form-group">
      <label>Confirme a senha*</label>
      <input
        id="validationCustom04"
        type="password"
        class="form-control"
        placeholder="Confirme a senha"
        v-model="user.senhaConfirmacao.value"
        @blur="progresso('senhaConfirmacao'); validaSenha()"
        required
      />
      <div v-if="validaSenha == false || user.senhaConfirmacao.value != ''" class="invalid-feedback">
        Senha incorreta!
      </div>
      <div v-else class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="form-group">
      <label>Data de nascimento*</label>
      <input
        id="validationCustom05"
        type="date"
        class="form-control"
        v-model="user.dataNascimento.value"
        @blur="progresso('dataNascimento')"
      />
      <div class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="form-group">
      <label>CEP*</label>
      <input
        id="validationCustom06"
        type="text"
        class="form-control"
        placeholder="00000-000"
        v-model="user.cep.value"
        @blur="buscarCep()"
        required
        maxlength="8"
        minlength="8"
      />
      <div class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="form-group">
      <label>Estado*</label>
      <select id="validationCustom07" name="" class="form-select" required v-model="user.estado.value">
        <option selected disabled value="">Selecionar...</option>
        <option v-for="estado of estados" :key="estado.uf" :value="estado.name">
          {{ estado.uf }}
        </option>
      </select>
      <div class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="form-group">
      <label>Cidade*</label>
      <input
        id="validationCustom08"
        type="text"
        class="form-control"
        placeholder="Cidade"
        v-model="user.cidade.value"
        required
      />
      <div class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="form-group">
      <label>Rua*</label>
      <input
        id="validationCustom09"
        type="text"
        class="form-control"
        placeholder="Rua"
        v-model="user.rua.value"
        required
      />
      <div class="invalid-feedback">Obrigatório</div>
    </div>
    <div class="row">
      <div class="form-group col-8 m-0">
        <label>Bairro*</label>
        <input
          id="validationCustom10"
          type="text"
          class="form-control"
          placeholder="Bairro"
          v-model="user.bairro.value"
          required
        />
        <div class="invalid-feedback">Obrigatório</div>
      </div>
      <div class="form-group col-4 m-0">
        <label>N°*</label>
        <input
          id="validationCustom11"
          type="text"
          class="form-control"
          placeholder="N°"
          v-model="user.num.value"
          @blur="progresso(user.num)"
          required
        />
        <div class="invalid-feedback">Obrigatório</div>
      </div>
    </div>
    <div class="mb-3">
      <label for="formFile" class="form-label">Foto de perfil - Opcional</label>
      <input class="form-control" type="file" id="formFile" @change="UploadImagem($event)" />
    </div>
    <div class="form-group">
      <label>Biografia - Opcional</label>
      <textarea
        name="biografia"
        id=""
        rows="8"
        class="form-control"
        placeholder="Escreva sobre você"
        v-model="user.biografia"
        @blur="progresso(user.biografia)"
      ></textarea>
    </div>
    <div class="form-group d-flex flex-column">
      <h2>Hobbies - Opcional</h2>
      <div class="m-3">
        <input
          class="form-check-input m-0"
          type="checkbox"
          v-model="user.hobbies"
          value="esporte"
        />
        <label for="" class="ms-2 checkbox">Esportes</label>
      </div>
      <div class="m-3">
        <input class="form-check-input m-0" type="checkbox" v-model="user.hobbies" value="música" />
        <label for="" class="ms-2 checkbox">Música</label>
      </div>
      <div class="m-3">
        <input
          class="form-check-input m-0"
          type="checkbox"
          v-model="user.hobbies"
          value="viagens"
        />
        <label for="" class="ms-2 checkbox">Viagens</label>
      </div>
      <div class="m-3">
        <input
          class="form-check-input m-0"
          type="checkbox"
          v-model="user.hobbies"
          value="leitura"
        />
        <label for="" class="ms-2 checkbox">Leitura</label>
      </div>
      <div class="m-3">
        <input class="form-check-input m-0" type="checkbox" v-model="user.hobbies" value="Outro" />
        <label for="" class="ms-2 checkbox">Outro</label>
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
          v-model="user.linguagemPref"
          value="javascript"
        />
        <label class="form-check-label ms-2 radio" for="flexRadioDefault1">Javascript</label>
      </div>
      <div class="form-check m-4">
        <input
          class="form-check-input m-0"
          type="radio"
          name="flexRadioDefault"
          id="flexRadioDefault1"
          v-model="user.linguagemPref"
          value="python"
        />
        <label class="form-check-label ms-2 radio" for="flexRadioDefault1"> Python</label>
      </div>
      <div class="form-check m-4">
        <input
          class="form-check-input m-0"
          type="radio"
          name="flexRadioDefault"
          id="flexRadioDefault1"
          v-model="user.linguagemPref"
          value="C"
        />
        <label class="form-check-label ms-2 radio" for="flexRadioDefault1">C</label>
      </div>
      <div class="form-check m-4">
        <input
          class="form-check-input m-0"
          type="radio"
          name="flexRadioDefault"
          id="flexRadioDefault1"
          v-model="user.linguagemPref"
          value="PHP"
        />
        <label class="form-check-label ms-2 radio" for="flexRadioDefault1">PHP</label>
      </div>
      <div class="form-check m-4">
        <input
          class="form-check-input m-0"
          type="radio"
          name="flexRadioDefault"
          id="flexRadioDefault1"
          v-model="user.linguagemPref"
          value="C++"
        />
        <label class="form-check-label ms-2 radio" for="flexRadioDefault1"> C++</label>
      </div>
      <div class="form-check m-4">
        <input
          class="form-check-input m-0"
          type="radio"
          name="flexRadioDefault"
          id="flexRadioDefault1"
          v-model="user.linguagemPref"
          value="outra"
        />
        <label class="form-check-label ms-2 radio" for="flexRadioDefault1"> Outra </label>
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

#titulo {
  width: 100%;
  display: flex;
  justify-content: center;
  margin-bottom: 4vh;
}

#perfilTitulo {
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

#fotoPerfil img {
  width: 150px;
  height: 150px;
  border-radius: 50%;
}

section {
  word-wrap: break-word;
}

@keyframes aparecer {
  0% {
    opacity: 0;
  }

  100% {
    opacity: 1;
  }
}

.contentInfo {
  animation: aparecer 1.8s;
}
.loading {
  width: 100%;
  height: 100%;
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
}
.corLilas {
  color: #7d5ff5;
}

#corRoxa {
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
