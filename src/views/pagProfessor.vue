<template>
  <!-- container do formulario -->
  <div class="square">
    <h2 class="titulo">Cadastro de Professores</h2>
    <!-- mensagens de erro -->
    <div v-if="showSuccessMessage" class="mensagem-sucesso">
      Dados salvos com sucesso!
    </div>
    <div v-if="showErrorMessage" class="mensagem-erro">
      Ocorreu um erro ao salvar os dados. Tente novamente!
    </div>
    <!-- segundo container formulario -->
    <div class="container-formulario">
      <div class="linha">
        <div class="item-input">
          <label for="name" class="campo-label">Nome:</label>
          <v-text-field
            id="name"
            v-model="name"
            outlined
            class="campo-input"
            :error-messages="nameErrorMessages"
            @blur="validateName"
            :placeholder="'Digite aqui'"
            :style="{ backgroundColor: inputBgColor, color: inputTextColor }"
          ></v-text-field>
        </div>
      </div>

      <div class="item-input">
        <label for="descricao" class="campo-label">Descrição:</label>
        <v-textarea
          id="descricao"
          v-model="descricao"
          outlined
          class="descri-input"
          rows="3"
          :placeholder="'Digite aqui'"
          :style="{ backgroundColor: inputBgColor, color: inputTextColor }"
        ></v-textarea>
      </div>

      <label class="campo-label">Selecione os Dias da Semana:</label>
      <v-alert
        v-if="selectedDaysErrorMessage.length > 0"
        type="error"
        class="error-message"
      >
        {{ selectedDaysErrorMessage }}
      </v-alert>

      <div class="dias-semana">
        <v-checkbox
          v-for="dia in dias"
          :key="dia.sigla"
          :label="dia.sigla"
          :value="dia.id"
          v-model="selectedDays"
          class="item-checkbox"
          @change="validateSelectedDays"
          :style="{ backgroundColor: checkboxBgColor, color: checkboxTextColor }"
        ></v-checkbox>
      </div>

      <div class="botoes">
        <v-btn @click="saveData" class="botao-acao-salvar">Salvar</v-btn>
      </div>

    </div>
  </div>
</template>

<script>
import { criarProfessor } from '@/API/service';
import { ref } from 'vue';

export default {
  setup() {
    const showSuccessMessage = ref(false);
    const showErrorMessage = ref('');
    const name = ref('');
    const descricao = ref('');
    const dias = ref([
      { sigla: 'Dom', id: 1, nome: 'domingo' },
      { sigla: 'Seg', id: 2, nome: 'segunda' },
      { sigla: 'Ter', id: 3, nome: 'terça' },
      { sigla: 'Qua', id: 4, nome: 'quarta' },
      { sigla: 'Qui', id: 5, nome: 'quinta' },
      { sigla: 'Sex', id: 6, nome: 'sexta' },
      { sigla: 'Sab', id: 7, nome: 'sábado' },
    ]);
    const selectedDays = ref([]);
    const nameErrorMessages = ref([]);
    const selectedDaysErrorMessage = ref('');
    const cancelDialog = ref(false);

    const inputBgColor = ref('#fafafa');
    const inputTextColor = ref('#2a3d73');
    const checkboxBgColor = ref('transparent');
    const checkboxTextColor = ref('#2a3d73');

    const validateName = () => {
      nameErrorMessages.value = [];
      if (!name.value) {
        nameErrorMessages.value.push('O nome é obrigatório.');
      }
    };

    const validateSelectedDays = () => {
      selectedDaysErrorMessage.value = '';
      if (selectedDays.value.length === 0) {
        selectedDaysErrorMessage.value = 'Pelo menos um dia deve ser selecionado.';
      }
    };

    const mapSelectedDaysToBooleans = () => {
      const selectedDaysBooleans = {
        domingo: false,
        segunda: false,
        terça: false,
        quarta: false,
        quinta: false,
        sexta: false,
        sábado: false,
      };

      selectedDays.value.forEach(id => {
        const diaSelecionado = dias.value.find(dia => dia.id === id);
        if (diaSelecionado) {
          selectedDaysBooleans[diaSelecionado.nome] = true;
        }
      });

      return selectedDaysBooleans;
    };

    const cadastrarProfessor = async () => {
      try {
        const professorData = {
          nome: name.value,
          descricao: descricao.value,
          ...mapSelectedDaysToBooleans()
        };
        await criarProfessor(professorData);
        showSuccessMessage.value = true;
        showErrorMessage.value = '';

        setTimeout(() => {
          showSuccessMessage.value = false;
        }, 3000);

      } catch (error) {
        console.error('Erro ao cadastrar professor:', error);
        
        if (error.response && error.response.data && error.response.data.message) {
          showErrorMessage.value = error.response.data.message;
        } else {
          showErrorMessage.value = 'Erro desconhecido ao cadastrar o professor.';
        }

        showSuccessMessage.value = false;
        setTimeout(() => {
          showErrorMessage.value = '';
        }, 3000);
      }
    };

    const saveData = async () => {
      validateName();
      validateSelectedDays();

      if (!name.value || selectedDays.value.length === 0) {
        return;
      }

      try {
        await cadastrarProfessor();
      } catch (error) {
        console.error('Erro ao salvar dados:', error);
      }
    };

    return {
      name,
      descricao,
      dias,
      selectedDays,
      nameErrorMessages,
      selectedDaysErrorMessage,
      showSuccessMessage,
      showErrorMessage,
      cancelDialog,
      inputBgColor,
      inputTextColor,
      checkboxBgColor,
      checkboxTextColor,
      validateName,
      validateSelectedDays,
      saveData,
    };
  },
};
</script>

<style scoped lang="sass">
.square
  display: flex
  flex-direction: column
  justify-content: center
  align-items: center
  margin: 20px auto
  padding: 20px
  max-width: 1200px
  width: 100%
  height: auto
  background-color: #f0f0f0 
  box-shadow: 2px 4px 10px rgba(0,0,0,0.2)
  position: relative
  border-radius: 5px

  .titulo
    color: #2a3d73
    font-weight: bold
    text-align: center
    font-size: 24px
    margin-bottom: 20px

  .container-formulario
    display: flex
    flex-direction: column
    gap: 20px
    width: 100%
    padding: 20px
    box-sizing: border-box

  .linha
    display: flex
    gap: 20px
    justify-content: space-between
    align-items: center
    flex-wrap: wrap

  .item-input
    flex: 1
    min-width: 250px

  .campo-input, .descri-input
    width: 100%
    font-size: 14px
    border-radius: 4px
    border: 1px solid #ccc
    padding: 10px
    transition: border-color 0.3s ease, box-shadow 0.3s ease
    background-color: #fafafa
    color: #2a3d73

  .dias-semana
    display: flex
    flex-wrap: wrap
    gap: 10px
    justify-content: center

  .botoes
    display: flex
    justify-content: flex-end
    gap: 10px
    margin-top: 20px

  .botao-acao-salvar
    transition: background-color 0.3s ease, transform 0.2s ease
    &:hover
      transform: scale(1.05)
      background-color: #10f448
      text-shadow: 0px 1px 5px rgba(255,255,255,1 )

  .mensagem-sucesso 
    margin-top: 20px
    color: green
    font-weight: bold
    text-align: center
    background: #10f448
    border-radius: 5px
    padding: 10px
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2)

  .mensagem-erro 
    margin-top: 20px
    color: red
    font-weight: bold
    text-align: center
    background: #991418
    border-radius: 5px
    padding: 10px
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2)

  .campo-label
    color: #2a3d73
    font-size: 14px
    font-weight: 400

  body, html
    height: 100%
    margin: 0

</style>
