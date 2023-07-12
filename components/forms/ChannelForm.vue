<template>
  <div class="choose_channels">
    <h2>Выберите каналы:</h2>
    <label v-for="(channel, index) in channels" :key="index">
      <input type="checkbox" v-model="selectedChannels" :value="channel">
      {{ getChannelName(channel) }}
    </label>

    <div class="selected_channels" v-for="channel in selectedChannels" :key="channel">
      <h2>{{ getChannelName(channel) }}</h2>
      <label class="message_text">
        Текст сообщения:
        <textarea v-model="messages[channel].text" :maxlength="getMaximumCharacters(channel)"></textarea>
      </label>
      <template v-if="getButtonSupport(channel)">
        <label class="keyboard_type">
          Клавиатура:
          <select v-model="messages[channel].keyboardType" @change="onChangeKeyboardType(channel)">
            <option v-for="option in keyboardTypeOptions" :value="option.value" :key="option.value">
              {{ option.label }}
            </option>
          </select>
        </label>
        <div class="buttons_number">
          <label>
            Количество кнопок:
            <input type="number" v-model="messages[channel].buttonCount" @input="updateButtonFields(channel)" min=0>
          </label>
        </div>
        <div class="button_info" v-for="(button, index) in messages[channel].buttons" :key="index">
          <label>
            Текст кнопки {{ index + 1 }}:
            <input type="text" v-model="button.text" @input="onButtonTextInput(channel, button.text)"
              :maxlength="getMaximumCharactersInButton(channel)">
          </label>
          <label>
            Тип кнопки:
            <select v-model="button.type" @change="onChangeButtonType(messages[channel].buttons, button, channel)">
              <option v-for="option in getSupportedButtonTypes(channel, messages[channel].keyboardType)"
                :value="option.value" :key="option.value">
                {{ option.label }}
              </option>
            </select>
          </label>
        </div>
      </template>
    </div>

    <button class="save_button" @click="saveSettings">Сохранить</button>
  </div>
</template>


<script>
import axios from 'axios';
export default {
  data() {
    return {
      channels: ['vk', 'telegram', 'whatsapp', 'sms'],
      selectedChannels: [],
      keyboardTypeOptions: [
        {
          value: 'standard',
          label: 'Стандартное отображение',
        },
        {
          value: 'inline',
          label: 'Inline-отображение',
        },
      ],
      buttonTypeOptions: [
        {
          value: 'quick_reply',
          label: 'Кнопка с быстрым ответом',
        },
        {
          value: 'link',
          label: 'Кнопка со ссылкой',
        },
      ],
      channelNames: {
        vk: 'ВКонтакте',
        telegram: 'Telegram',
        whatsapp: 'WhatsApp',
        sms: 'SMS',
      },
      maximumCharachters: {
        vk: 4096,
        telegram: 4096,
        whatsapp: 1000,
        sms: -1,
      },
      maximumButtons: {
        vk: 40,
        telegram: Infinity,
        whatsapp: 10,
        sms: -1,
      },
      maximumCharachtersInButton: {
        vk: 40,
        telegram: 64,
        whatsapp: 20,
        sms: -1,
      },
      buttonsSupport: {
        vk: true,
        telegram: true,
        whatsapp: true,
        sms: false,
      },
      linkStandartSupport: {
        vk: true,
        telegram: false,
        whatsapp: false,
        sms: false,
      },
      linkInlineSupport: {
        vk: true,
        telegram: true,
        whatsapp: true,
        sms: false,
      },
      maxLinkInlineSupport: {
        vk: Infinity,
        telegram: Infinity,
        whatsapp: 1,
        sms: -1,
      },
      maxInlineButtons: {
        vk: 10,
        telegram: Infinity,
        whatsapp: 3,
        sms: -1,
      },
      messages: {},
    };
  },
  methods: {
    onChangeButtonType(buttons, button, channel) {
      const linkCount = buttons.filter((button) => button.type === 'link').length;
      if (linkCount > this.getMaxLinkInlineSupport(channel)) {
        alert("Превышено максимальное количество ссылок для inline-клавиатуры")
        button.type = 'quick_reply'
      }
    },
    getSupportedButtonTypes(channel, keyboardType) {
      let options = [...this.buttonTypeOptions];
      if (keyboardType === 'standard' && !this.getLinkStandardSupport(channel)) {
        options = options.filter(option => option.value !== 'link');
      } else if (keyboardType === 'inline' && !this.getLinkInlineSupport(channel)) {
        options = options.filter(option => option.value !== 'link');
      }
      return options;
    },
    onButtonTextInput(channel, text) {
      if (text.length > this.getMaximumCharactersInButton(channel)) {
        alert(`Превышено максимальное количество символов для кнопки (${this.getMaximumCharactersInButton(channel)})`)
      }
    },
    onChangeKeyboardType(channel) {
      this.updateButtonFields(channel);
    },
    getButtonSupport(channel) {
      return this.buttonsSupport[channel];
    },
    getChannelName(channel) {
      return this.channelNames[channel];
    },
    getMaximumCharacters(channel) {
      return this.maximumCharachters[channel];
    },
    getLinkStandardSupport(channel) {
      return this.linkStandartSupport[channel];
    },
    getLinkInlineSupport(channel) {
      return this.linkInlineSupport[channel];
    },
    getMaxLinkInlineSupport(channel) {
      return this.maxLinkInlineSupport[channel];
    },
    getMaximumButtons(channel) {
      return this.maximumButtons[channel];
    },
    getMaximumCharactersInButton(channel) {
      return this.maximumCharachtersInButton[channel];
    },
    getMaximumInlineButtons(channel) {
      return this.maxInlineButtons[channel];
    },
    getButtonSupport(channel) {
      return this.buttonsSupport[channel];
    },
    saveSettings() {
      console.log("MESSAGES", this.messages)
      // создание объекта настроек
      const settings = Object.entries(this.messages).map(([channel, properties]) => ({
        channel,
        ...properties
      }));

      // Результат
      const result = { settings }

      console.log(result)

      let config = {
        method: 'post',
        maxBodyLength: Infinity,
        url: 'http://localhost:4000/api/settings',
        headers: {},
        data: result
      };

      axios.request(config)
        .then((response) => {
          alert(`${response.data.message}`);
        })
        .catch((error) => {
          alert(`Ошибка: ${error}`)
        });
    },
    updateButtonFields(channel) {
      let buttonCount = this.messages[channel].buttonCount;
      const currentButtonCount = this.messages[channel].buttons.length;

      if (this.messages[channel].keyboardType === 'inline') {
        if (buttonCount > this.getMaximumInlineButtons(channel)) {
          alert(`Превышено максимальное количество кнопок для inline-клавиатуры (${this.getMaximumInlineButtons(channel)})`)
          this.messages[channel].buttonCount = this.getMaximumInlineButtons(channel)
        }
      } else {
        if (buttonCount > this.getMaximumButtons(channel)) {
          alert(`Превышено максимальное количество кнопок для стандартной клавиатуры (${this.getMaximumButtons(channel)})`)
          this.messages[channel].buttonCount = this.getMaximumButtons(channel)
        }
      }

      buttonCount = this.messages[channel].buttonCount;

      if (buttonCount > currentButtonCount) {
        // Добавление новых текстовых полей для кнопок
        const diff = buttonCount - currentButtonCount;
        for (let i = 0; i < diff; i++) {
          this.messages[channel].buttons.push({ text: '', type: 'quick_reply' });
        }
      } else if (buttonCount < currentButtonCount) {
        // Удаление лишних текстовых полей для кнопок
        this.messages[channel].buttons.splice(buttonCount);
      }
    },
  },
  watch: {
    selectedChannels: {
      immediate: true,
      handler(newSelectedChannels) {
        // Создание копии предыдущего состояния объекта messages
        const previousMessages = { ...this.messages };

        // Инициализация объекта только для новых каналов
        this.messages = newSelectedChannels.reduce((acc, channel) => {
          if (previousMessages.hasOwnProperty(channel)) {
            // Если канал уже существует, сохраняем предыдущие значения
            acc[channel] = { ...previousMessages[channel] };
          } else {
            // Если не имеет поддержку кнопок, инициализируем только текст
            if (!this.getButtonSupport(channel)) {
              acc[channel] = {
                text: '',
              };
            }
            else
              // Если канал новый, инициализируем значения
              acc[channel] = {
                text: '',
                keyboardType: 'standard',
                buttonCount: 1,
                buttons: Array.from({ length: 1 }, () => ({ text: '', type: 'quick_reply' })),
              };
          }
          return acc;
        }, {});
      },
    },
  },
};
</script>

<style>
@import "../../assets/css/channel-form.css";
</style>