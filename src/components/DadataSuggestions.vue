<template>
  <input
      v-model="model"
      @input="onInput"
      class="dadata-input"
      type="text"
      autocomplete="off"
      @change="onChange"
  >
</template>

<script>
import * as $ from 'jquery';
import '../dadata.js';

export default {
  name: 'DadataSuggestions',
  pluginOptions: {},
  props: {
    value: {
      type: String,
      default: '',
    },
    /** Поле для хранения полной информации о результате */
    fullInfo: {},
    /** Параметры для Дадата */
    options: {},
    /** API-ключ */
    token: {
      type: String,
      default: '',
    },
    /** Тип подсказок */
    type: {
      type: String,
      default: '',
    },
    fieldValue: {
      type: String,
      default: 'value',
    },
  },
  data() {
    return {
      model: '',
      currentOptions: {
        /** API-ключ */
        token: '',
        /** Тип подсказок */
        type: '',

        /** ИДЕНТИФИКАТОР_РАЗРАБОТЧИКА */
        partner: ''
      },
      defaultOptions: {
        /** Прокручивать текстовое поле к верхней границе экрана при фокусе */
        scrollOnFocus: false,
        /** Автоматически подставлять подходящую подсказку из списка, когда текстовое поле теряет фокус.*/
        triggerSelectOnBlur: false,
        /** Автоматически подставлять подходящую подсказку из списка при нажатии на Enter. */
        triggerSelectOnEnter: true,
        /** Что показывать в правом углу текстового поля подсказок */
        addon: 'spinner',
      },
    };
  },
  created() {
    this.parseOptions()
    if (!this.currentOptions.token) {
      console.warn('Dadata: Необходимо указать API-токен');
    }

    if (!this.currentOptions.type) {
      console.warn('Dadata: Необходимо указать тип подсказок');
    }
  },

  mounted() {
    this.model = this.value;
    this.initSuggestion();
  },

  methods: {
    parseOptions() {
      this.currentOptions = { ...this.currentOptions, ...this.defaultOptions };
      this.currentOptions = { ...this.currentOptions, ...this.$options.pluginOptions };
      this.currentOptions = { ...this.currentOptions, ...this.options };
      if (this.token) this.currentOptions.token = this.token;
      if (this.type) this.currentOptions.type = this.type;

      this.currentOptions = Object.assign(this.currentOptions, {
        onSelect: suggestion => {
          this.$emit('update:fullInfo', suggestion);
          this.$emit('change', suggestion);
          this.onSelect(suggestion);
        },
      });
      this.currentOptions = Object.assign(this.currentOptions, {
        onSearchError: (q, w, e, r, t) => {
          if (w.status === 403) {
            console.warn('Ошибка доступа! \n ' + w.responseJSON.message)
          }
        },
      });

      this.currentOptions.partner = 'VUE.76941';
    },

    /**
     * Инициализация модуля
     */
    initSuggestion() {
      $(this.$el).suggestions(this.currentOptions);
    },

    destroySuggestion() {
      const plugin = $(this.$el).suggestions();
      plugin.dispose();
    },

    /**
     * Выбор элемента из списка
     */
    async onSelect(suggestion) {
      this.model = null
      await this.$nextTick();
      if (this.fieldValue === 'value' || this.fieldValue === 'unrestricted_value') {
        this.model = suggestion[this.fieldValue];
      } else {
        let result = suggestion.data;
        this.fieldValue.split('.').forEach((item, index) => {
          result = result[item];
        })
        this.model = result;
      }
      this.$emit('input', this.model);
    },

    onChange() {
      if (this.model === null || this.model === '') {
        this.$emit('update:fullInfo', null);
      } else {
        if (this.model !== this.fullInfo?.value) {
          this.$emit('update:fullInfo', null);
        }
      }
      this.$emit('input', this.model);
      this.$emit('change', this.model);
    },
    
    onInput() {
      this.$emit('input', this.model);
      this.$emit('update:fullInfo', null);
    },

  },

  destroyed() {
    this.destroySuggestion();
  },

  watch: {
    value() {
      this.model = this.value;
    },
  },
};
</script>

<style>
.dadata-input {
  border-radius: 5px;
  padding: 10px 15px;
  border: 1px solid rgba(29, 29, 27, 0.3);
  transition: all 0.2s ease;
}

.suggestions-nowrap {
  white-space: nowrap
}

.suggestions-input {
  -ms-box-sizing: border-box;
  -moz-box-sizing: border-box;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  width: 100%
}

.suggestions-input::-ms-clear {
  display: none
}

.suggestions-wrapper {
  position: relative;
  margin: 0;
  padding: 0;
  vertical-align: top;
  -webkit-text-size-adjust: 100%
}

.suggestions-addon {
  display: none;
  opacity: 0;
  position: absolute
}

.suggestions-addon[data-addon-type=spinner] {
  background: rgba(255, 255, 255, .75) url(data:image/gif;base64,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) 50% 50% no-repeat;
  -webkit-border-radius: 50%;
  -moz-border-radius: 50%;
  border-radius: 50%
}

.suggestions-addon[data-addon-type=clear] {
  background: transparent url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0nMTAwJyBoZWlnaHQ9JzEwMCcgeG1sbnM9J2h0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnJz48Zz48cGF0aCBkPSdtMjAsMjBtMTAsMGwyMCwyMGwyMCwtMjBsMTAsMTBsLTIwLDIwbDIwLDIwbC0xMCwxMGwtMjAsLTIwbC0yMCwyMGwtMTAsLTEwbDIwLC0yMGwtMjAsLTIwbDEwLC0xMCcgZmlsbD0nIzk5OScvPjwvZz48L3N2Zz4=) 50% 50% no-repeat;
  background-size: contain;
  cursor: pointer;
  opacity: .8
}

.suggestions-addon[data-addon-type=clear]:hover {
  opacity: 1
}

.suggestions-suggestions {
  background: #fff;
  border: 1px solid #999;
  -ms-box-sizing: border-box;
  -moz-box-sizing: border-box;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  cursor: default;
  z-index: 9999;
  -webkit-text-size-adjust: 100%
}

.suggestions-suggestions strong {
  font-weight: 400;
  color: #39f
}

.suggestions-suggestions.suggestions-mobile {
  border-style: none
}

.suggestions-suggestions.suggestions-mobile .suggestions-suggestion {
  border-bottom: 1px solid #ddd
}

.suggestions-suggestion {
  padding: 4px 4px;
  overflow: hidden
}

.suggestions-suggestion:hover {
  background: #f7f7f7
}

.suggestions-selected {
  background: #f0f0f0
}

.suggestions-selected:hover {
  background: #f0f0f0
}

.suggestions-hint {
  padding: 4px 4px;
  white-space: nowrap;
  overflow: hidden;
  color: #777;
  font-size: 85%;
  line-height: 20px
}

.suggestions-constraints {
  list-style: none !important;
  margin: 0 !important;
  padding: 0 !important;
  position: absolute !important;
  white-space: nowrap !important
}

.suggestions-constraints:empty {
  display: none !important
}

.suggestions-constraints li {
  background: #f8f8f8;
  border: 1px solid #ccc;
  -webkit-border-radius: 3px;
  -moz-border-radius: 3px;
  border-radius: 3px;
  cursor: default;
  display: inline-block;
  margin: 0 4px 0 0;
  padding: 0 .5em
}

.suggestions-constraints li .suggestions-remove {
  height: 1em;
  width: 1em;
  display: inline-block;
  margin: 0 0 0 .25em;
  background: transparent url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0nMTAwJyBoZWlnaHQ9JzEwMCcgeG1sbnM9J2h0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnJz48Zz48cGF0aCBkPSdtMjAsMjBtMTAsMGwyMCwyMGwyMCwtMjBsMTAsMTBsLTIwLDIwbDIwLDIwbC0xMCwxMGwtMjAsLTIwbC0yMCwyMGwtMTAsLTEwbDIwLC0yMGwtMjAsLTIwbDEwLC0xMCcgZmlsbD0nIzk5OScvPjwvZz48L3N2Zz4=) 50% 50% no-repeat;
  background-size: contain;
  cursor: pointer;
  opacity: .8
}

.suggestions-constraints li .suggestions-remove:hover {
  opacity: 1
}

.suggestions-constraints li span {
  vertical-align: middle
}

.suggestions-subtext {
  color: #777;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis
}

.suggestions-subtext_inline {
  display: inline-block;
  min-width: 6em;
  vertical-align: bottom;
  margin: 0 .5em 0 0
}

.suggestions-subtext-delimiter {
  display: inline-block;
  width: 2px
}

.suggestions-subtext_label {
  margin: 0 0 0 .25em;
  -webkit-border-radius: 3px;
  -moz-border-radius: 3px;
  border-radius: 3px;
  padding: 0 3px;
  background: #f5f5f5;
  font-size: 85%
}

.suggestions-value[data-suggestion-status=LIQUIDATED] {
  position: relative
}

.suggestions-value[data-suggestion-status=LIQUIDATED]:after {
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  border-top: 1px solid rgba(0, 0, 0, .4);
  content: ""
}

.suggestions-promo {
  font-size: 85%;
  display: none;
  color: #777;
  padding: 4px;
  text-align: center
}

.suggestions-promo a {
  color: #777;
  display: block;
  filter: grayscale(100%);
  line-height: 20px;
  text-decoration: none
}

.suggestions-promo a:hover {
  filter: grayscale(0)
}

.suggestions-promo svg {
  height: 20px;
  vertical-align: bottom
}

.suggestions-promo-desktop {
  position: absolute;
  top: 0;
  right: 0;
  text-align: left
}
</style>
