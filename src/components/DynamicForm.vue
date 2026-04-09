<template>
  <div class="dynamic-form-wrapper">
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons+Round" rel="stylesheet" />
    <form @submit.prevent="handleSubmit" class="dynamic-form" novalidate>
      <div
        v-for="field in fields"
        :key="field.name"
        class="form-group"
        :class="{ 'has-error': errors[field.name], 'full-width': field.fullWidth }"
      >
        <label
          v-if="!['checkbox', 'toggle','divider'].includes(field.type)"
          :for="field.name"
          class="form-label"
        >
          {{ field.label }}
          <span v-if="field.required" class="required-asterisk">*</span>
          <span v-if="field.hint" class="field-hint">{{ field.hint }}</span>
        </label>

        <div
          v-if="['text', 'email', 'number'].includes(field.type)"
          class="input-wrapper"
          :class="{ 'has-icon': field.icon }"
        >
          <span v-if="field.icon" class="input-icon material-icons-round">{{ field.icon }}</span>
          <InputText
            :id="field.name"
            :type="field.type"
            v-model="formData[field.name]"
            :required="field.required"
            :placeholder="field.placeholder"
            :min="field.min"
            :max="field.max"
            :step="field.step"
            class="custom-input"
            @blur="validateField(field)"
          />
        </div>

        <div v-else-if="field.type === 'password'" class="input-wrapper password-wrapper">
          <span class="input-icon material-icons-round">lock</span>
          <InputText
            :id="field.name"
            :type="passwordVisible[field.name] ? 'text' : 'password'"
            v-model="formData[field.name]"
            :required="field.required"
            :placeholder="field.placeholder || 'Enter password'"
            class="custom-input"
            @blur="validateField(field)"
          />
          <button
            type="button"
            class="password-toggle"
            @click="togglePasswordVisibility(field.name)"
            :aria-label="passwordVisible[field.name] ? 'Hide password' : 'Show password'"
          >
            <span class="material-icons-round eye-icon">
              {{ passwordVisible[field.name] ? 'visibility_off' : 'visibility' }}
            </span>
          </button>
        </div>

        <div v-else-if="field.type === 'textarea'" class="input-wrapper textarea-wrapper">
          <span v-if="field.icon" class="input-icon textarea-icon material-icons-round">{{ field.icon }}</span>
          <Textarea
            :id="field.name"
            v-model="formData[field.name]"
            :required="field.required"
            :placeholder="field.placeholder"
            :rows="field.rows || 4"
            :maxlength="field.maxLength"
            class="custom-input custom-textarea"
            :autoResize="field.autoResize !== false"
            @blur="validateField(field)"
          />
          <div v-if="field.maxLength" class="char-counter">
            {{ (formData[field.name] || '').length }} / {{ field.maxLength }}
          </div>
        </div>

        <Select
          v-else-if="field.type === 'select'"
          :id="field.name"
          v-model="formData[field.name]"
          :options="field.options"
          optionLabel="label"
          optionValue="value"
          :placeholder="field.placeholder || 'Select an option'"
          class="custom-input custom-dropdown"
          @change="validateField(field)"
        />

        <MultiSelect
          v-else-if="field.type === 'multiselect'"
          :id="field.name"
          v-model="formData[field.name]"
          :options="field.options"
          optionLabel="label"
          optionValue="value"
          :placeholder="field.placeholder || 'Select options'"
          :maxSelectedLabels="field.maxSelectedLabels || 3"
          class="custom-input custom-dropdown"
          @change="validateField(field)"
        />

        <div v-else-if="field.type === 'radio'" class="radio-group">
          <div
            v-for="option in field.options"
            :key="option.value"
            class="radio-option"
            :class="{ selected: formData[field.name] === option.value }"
            @click="formData[field.name] = option.value; validateField(field)"
          >
            <RadioButton
              :inputId="`${field.name}_${option.value}`"
              :name="field.name"
              :value="option.value"
              v-model="formData[field.name]"
            />
            <label :for="`${field.name}_${option.value}`" class="radio-label">
              <span v-if="option.icon" class="option-icon material-icons-round">{{ option.icon }}</span>
              {{ option.label }}
            </label>
          </div>
        </div>

        <div v-else-if="field.type === 'range'" class="range-wrapper">
          <div class="range-values">
            <span class="range-min">{{ field.min ?? 0 }}</span>
            <span class="range-current">{{ formData[field.name] ?? field.min ?? 0 }}</span>
            <span class="range-max">{{ field.max ?? 100 }}</span>
          </div>
          <Slider
            :id="field.name"
            v-model="formData[field.name]"
            :min="field.min ?? 0"
            :max="field.max ?? 100"
            :step="field.step ?? 1"
            class="custom-slider"
          />
          <div v-if="field.labels" class="range-labels">
            <span v-for="(lbl, i) in field.labels" :key="i">{{ lbl }}</span>
          </div>
        </div>

        <div
          v-else-if="['date', 'time', 'datetime-local', 'month', 'week'].includes(field.type)"
          class="input-wrapper"
        >
          <span v-if="field.icon" class="input-icon material-icons-round">{{ field.icon }}</span>
          <DatePicker
            v-if="field.type === 'date'"
            :id="field.name"
            v-model="formData[field.name]"
            :required="field.required"
            :placeholder="field.placeholder || 'dd/mm/yyyy'"
            dateFormat="dd/mm/yy"
            :showIcon="true"
            class="custom-input custom-calendar"
            @blur="validateField(field)"
          />
          <InputText
            v-else
            :id="field.name"
            :type="field.type"
            v-model="formData[field.name]"
            :required="field.required"
            class="custom-input"
            @blur="validateField(field)"
          />
        </div>

        <div v-else-if="field.type === 'color'" class="color-wrapper">
          <ColorPicker
            :id="field.name"
            v-model="formData[field.name]"
            format="hex"
            class="custom-color"
          />
          <span class="color-value">#{{ formData[field.name] || '000000' }}</span>
        </div>

        <div v-else-if="field.type === 'file'" class="file-wrapper">
          <label :for="field.name" class="file-label">
            <span class="material-icons-round file-icon">attach_file</span>
            <span class="file-text">{{ fileNames[field.name] || (field.placeholder || 'Choose a file') }}</span>
          </label>
          <input
            :id="field.name"
            type="file"
            :accept="field.accept"
            :multiple="field.multiple"
            class="file-input-hidden"
            @change="handleFileChange($event, field)"
          />
        </div>

        <div v-else-if="field.type === 'checkbox'" class="checkbox-group">
          <Checkbox
            :inputId="field.name"
            v-model="formData[field.name]"
            :binary="true"
            class="custom-checkbox"
          />
          <label :for="field.name" class="checkbox-label">
            {{ field.label }}
            <span v-if="field.required" class="required-asterisk">*</span>
          </label>
        </div>

        <div v-else-if="field.type === 'toggle'" class="toggle-group">
          <ToggleSwitch
            :inputId="field.name"
            v-model="formData[field.name]"
            class="custom-toggle"
          />
          <label :for="field.name" class="toggle-label">
            {{ field.label }}
            <span v-if="field.required" class="required-asterisk">*</span>
          </label>
        </div>

        <div v-else-if="field.type === 'tags'" class="tags-wrapper">
          <div class="tags-container">
            <span
              v-for="(tag, idx) in (formData[field.name] || [])"
              :key="idx"
              class="tag-chip"
            >
              {{ tag }}
              <button type="button" class="tag-remove" @click="removeTag(field.name, idx)">
                <span class="material-icons-round">close</span>
              </button>
            </span>
            <input
              :placeholder="field.placeholder || 'Add a tag...'"
              class="tag-input"
              @keydown.enter.prevent="addTag($event, field.name)"
              @keydown.comma.prevent="addTag($event, field.name)"
            />
          </div>
        </div>

        <div v-else-if="field.type === 'divider'" class="form-divider">
          <span v-if="field.label" class="divider-label">{{ field.label }}</span>
        </div>

        <Transition name="slide-error">
          <p v-if="errors[field.name]" class="error-message">
            <span class="material-icons-round">warning_amber</span>
            {{ errors[field.name] }}
          </p>
        </Transition>
      </div>

      <Button
        type="submit"
        :label="submitText"
        :loading="loading"
        class="submit-btn"
        :disabled="loading"
      />
    </form>
  </div>
</template>

<script>
import InputText from 'primevue/inputtext';
import Textarea from 'primevue/textarea';
import Select from 'primevue/select';
import MultiSelect from 'primevue/multiselect';
import Checkbox from 'primevue/checkbox';
import RadioButton from 'primevue/radiobutton';
import ToggleSwitch from 'primevue/toggleswitch';
import Slider from 'primevue/slider';
import DatePicker from 'primevue/datepicker';
import ColorPicker from 'primevue/colorpicker';
import Button from 'primevue/button';

export default {
  name: 'DynamicForm',

  components: {
    InputText,
    Textarea,
    Select,
    MultiSelect,
    Checkbox,
    RadioButton,
    ToggleSwitch,
    Slider,
    DatePicker,
    ColorPicker,
    Button,
  },

  props: {
    fields: {
      type: Array,
      required: true,
    },
    submitText: {
      type: String,
      default: 'Submit',
    },
    loading: {
      type: Boolean,
      default: false,
    },
  },

  emits: ['submit-form'],

  data() {
    return {
      formData: {},
      errors: {},
      passwordVisible: {},
      fileNames: {},
    };
  },

  created() {
    this.fields.forEach((field) => {
      if (field.type === 'checkbox' || field.type === 'toggle') {
        this.formData[field.name] = field.defaultValue ?? false;
      } else if (field.type === 'multiselect' || field.type === 'tags') {
        this.formData[field.name] = field.defaultValue ?? [];
      } else if (field.type === 'range') {
        this.formData[field.name] = field.defaultValue ?? field.min ?? 0;
      } else if (field.type === 'color') {
        this.formData[field.name] = field.defaultValue ?? 'ffffff';
      } else {
        this.formData[field.name] = field.defaultValue ?? null;
      }

      if (field.type === 'password') {
        this.passwordVisible[field.name] = false;
      }
    });
  },

  methods: {
    togglePasswordVisibility(name) {
      this.passwordVisible[name] = !this.passwordVisible[name];
    },

    handleFileChange(event, field) {
      const files = event.target.files;
      if (!files.length) return;
      this.formData[field.name] = field.multiple ? Array.from(files) : files[0];
      this.fileNames[field.name] = field.multiple
        ? `${files.length} files selected`
        : files[0].name;
    },

    addTag(event, name) {
      const value = event.target.value.trim().replace(/,$/, '');
      if (!value) return;
      if (!this.formData[name]) this.formData[name] = [];
      if (!this.formData[name].includes(value)) {
        this.formData[name] = [...this.formData[name], value];
      }
      event.target.value = '';
    },

    removeTag(name, index) {
      this.formData[name] = this.formData[name].filter((_, i) => i !== index);
    },

    validateField(field) {
      const value = this.formData[field.name];
      let error = '';

      if (field.required) {
        const isEmpty =
          value === null ||
          value === undefined ||
          value === '' ||
          (Array.isArray(value) && value.length === 0);
        if (isEmpty) {
          error = field.errorMessage || `${field.label} is required`;
        }
      }

      if (!error && field.type === 'email' && value) {
        const emailRe = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRe.test(value)) error = 'Please enter a valid email address';
      }

      if (!error && field.type === 'number' && value !== null && value !== '') {
        if (field.min !== undefined && Number(value) < field.min)
          error = `Minimum value is ${field.min}`;
        if (field.max !== undefined && Number(value) > field.max)
          error = `Maximum value is ${field.max}`;
      }

      if (!error && field.minLength && value && value.length < field.minLength) {
        error = `Minimum ${field.minLength} characters required`;
      }

      if (!error && field.pattern && value) {
        const re = new RegExp(field.pattern);
        if (!re.test(value)) error = field.patternMessage || 'Invalid format';
      }

      if (!error && field.validate) {
        error = field.validate(value, this.formData) || '';
      }

      this.errors[field.name] = error;
      return !error;
    },

    handleSubmit() {
      let valid = true;
      this.fields.forEach((field) => {
        if (field.type !== 'divider') {
          if (!this.validateField(field)) valid = false;
        }
      });
      if (valid) this.$emit('submit-form', { ...this.formData });
    },
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:ital,opsz,wght@0,9..40,300;0,9..40,400;0,9..40,500;0,9..40,600;1,9..40,300&family=DM+Mono:wght@400;500&display=swap');

*, *::before, *::after { box-sizing: border-box; }

.dynamic-form-wrapper {
  font-family: inherit;
  background: var(--background);
  padding: 2.5rem;
  border-radius: 20px;
  max-width: 560px;
  margin: 0 auto;
  position: relative;
  overflow: hidden;
  box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px, rgba(0, 0, 0, 0.3) 0px 7px 13px -3px, rgba(0, 0, 0, 0.2) 0px -3px 0px inset;
  --p-primary-color: var(--primary);
  --p-primary-color-text: var(--background);
  --p-focus-ring-color: var(--primary);
  --p-focus-ring-shadow: 0 0 0 3px color-mix(in srgb, var(--primary) 18%, transparent);
}

.dynamic-form {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.25rem 1.5rem;
  position: relative;
  z-index: 1;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.form-group.full-width,
.form-group:has(.submit-btn),
.form-group:has(.form-divider),
.form-group:has(.radio-group),
.form-group:has(.tags-wrapper),
.form-group:has(.custom-textarea) {
  grid-column: 1 / -1;
}

.form-label {
  font-size: 0.8rem;
  font-weight: 600;
  color: hsl(from var(--foreground) h s calc( l * 1.5 ));
  letter-spacing: 0.04em;
  text-transform: uppercase;
  display: flex;
  align-items: center;
  gap: 0.4rem;
}

.required-asterisk {
  color: var(--error);
  font-size: 0.85em;
}

.field-hint {
  font-weight: 400;
  text-transform: none;
  letter-spacing: 0;
  color: hsl(from var(--foreground) h s calc( l * 0.7 ));
  font-size: 0.75rem;
  margin-left: auto;
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.input-icon {
  position: absolute;
  left: 0.85rem;
  font-size: 1.1rem;
  pointer-events: none;
  z-index: 2;
  opacity: 0.45;
  color: hsl(from var(--foreground) h s calc( l * 0.7 ));
  user-select: none;
}

.input-wrapper.has-icon :deep(.custom-input),
.input-wrapper.password-wrapper :deep(.custom-input) {
  padding-left: 2.6rem;
}

:deep(.custom-input) {
  width: 100%;
  background: color-mix(in srgb, var(--background) 60%);
  color: var(--foreground);
  border: 1px solid color-mix(in srgb, var(--foreground) 60%, transparent);
  border-radius: 10px;
  padding: 0.7rem 0.9rem;
  font-size: 0.9rem;
  font-family: inherit;
  transition: border-color 0.2s, box-shadow 0.2s, background 0.2s;
  -webkit-appearance: none;
}

:deep(.custom-input::placeholder) {
  color: color-mix(in srgb, var(--foreground) 30%, transparent);
}

:deep(.custom-input:focus),
:deep(.custom-input:enabled:focus) {
  outline: none;
  border-color: var(--primary);
  background: color-mix(in srgb, var(--background) 30%);
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--primary) 18%, transparent);
}

.has-error :deep(.custom-input) {
  border-color: var(--error);
}

.password-wrapper :deep(.custom-input) {
  padding-right: 3rem;
}

.password-toggle {
  position: absolute;
  right: 0.6rem;
  background: none;
  border: none;
  cursor: pointer;
  padding: 0.3rem;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.15s, color 0.15s;
  z-index: 3;
  color: hsl(from var(--foreground) h s calc( l * 1.1 ));
}

.password-toggle:hover {
  background: color-mix(in srgb, var(--primary) 12%, transparent);
  color: var(--foreground);
}

.eye-icon {
  font-size: 1.15rem;
  user-select: none;
}

.textarea-wrapper {
  flex-direction: column;
  align-items: stretch;
}

.textarea-wrapper .input-icon.textarea-icon {
  position: static;
  padding: 0.5rem 0 0 0.5rem;
  align-self: flex-start;
}

:deep(.custom-textarea) {
  resize: vertical;
  min-height: 100px;
  padding: 0.7rem 0.9rem;
  line-height: 1.6;
}

.char-counter {
  align-self: flex-end;
  font-size: 0.72rem;
  color: hsl(from var(--foreground) h s calc( l * 0.7 ));
  font-family: inherit;
  margin-top: 0.25rem;
}

:deep(.custom-dropdown) {
  width: 100%;
  border-radius: 10px;
}

:deep(.custom-dropdown .p-select-label),
:deep(.custom-dropdown .p-multiselect-label) {
  font-family: inherit;
  font-size: 0.9rem;
  color: var(--foreground);
}

.radio-group {
  display: flex;
  flex-wrap: wrap;
  gap: 0.6rem;
}

.radio-option {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.6rem 1rem;
  border-radius: 10px;
  border: 1px solid color-mix(in srgb, var(--foreground) 60%, transparent);
  background: color-mix(in srgb, var(--background) 60%);
  cursor: pointer;
  transition: border-color 0.2s, background 0.2s, box-shadow 0.2s;
}

.radio-option:hover {
  border-color: color-mix(in srgb, var(--primary) 60%, transparent);
  background: color-mix(in srgb, var(--primary) 5%, transparent);
}

.radio-option.selected {
  border-color: var(--primary);
  background: color-mix(in srgb, hsl(from var(--primary) h s calc( l * 0.7 )) 10%, transparent);
  box-shadow: 0 0 0 1px hsl(from var(--primary) h s calc( l * 0.7 ));
}

.radio-label {
  font-size: 0.88rem;
  color: var(--foreground, #e2e4ff);
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.35rem;
}

.option-icon {
  font-size: 1.05rem;
  color: var(--foreground);
  opacity: 0.8;
  user-select: none;
}

.range-wrapper { display: flex; flex-direction: column; gap: 0.5rem; }

.range-values {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.range-min, .range-max {
  font-size: 0.72rem;
  color: color-mix(in srgb, var(--foreground) 80%);
  font-family: inherit;
}

.range-current {
  font-size: 1.1rem;
  font-weight: 700;
  color: var(--foreground);
  font-family: inherit;
}

:deep(.custom-slider .p-slider-range) {
  background: var(--primary);
}

:deep(.custom-slider .p-slider-handle) {
  border-color: var(--primary);
  background: var(--background);
  width: 1.2rem;
  height: 1.2rem;
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--primary) 10%, transparent);
  transition: box-shadow 0.2s;
}

:deep(.custom-slider .p-slider-handle:hover),
:deep(.custom-slider .p-slider-handle:focus) {
  box-shadow: 0 0 0 5px color-mix(in srgb, var(--primary) 20%, transparent);
}

.range-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.7rem;
  color: color-mix(in srgb, var(--foreground) 75%, transparent);
}

:deep(.custom-calendar) { width: 100%; }

:deep(.custom-calendar .p-inputtext) {
  width: 100%;
  background: color-mix(in srgb, var(--background) 60%);
  color: var(--foreground);
  border: 1px solid color-mix(in srgb, var(--foreground) 60%, transparent);
  border-radius: 10px;
  font-family: inherit;
  font-size: 0.9rem;
}

.color-wrapper {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.color-value {
  font-family: inherit;
  font-size: 0.85rem;
  color: color-mix(in srgb, var(--foreground) 70%, transparent);
  text-transform: uppercase;
}

.file-input-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  opacity: 0;
  overflow: hidden;
}

.file-label {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  padding: 0.7rem 1rem;
  border: 1.5px dashed color-mix(in srgb, var(--foreground) 60%, transparent);
  border-radius: 10px;
  cursor: pointer;
  font-size: 0.88rem;
  color: color-mix(in srgb, var(--foreground) 70%, transparent);
  background: color-mix(in srgb, var(--background) 50%);
  transition: border-color 0.2s, color 0.2s, background 0.2s;
  width: 100%;
}

.file-label:hover {
  border-color: var(--foreground);
  color: var(--primary);
}

.file-icon {
  font-size: 1.1rem;
  user-select: none;
}

.checkbox-group {
  display: flex;
  align-items: center;
  gap: 0.7rem;
  padding: 0.4rem 0;
}

.checkbox-label {
  font-size: 0.9rem;
  color: var(--foreground);
  cursor: pointer;
}

:deep(.custom-checkbox .p-checkbox-box.p-highlight) {
  background: var(--foreground);
  border-color: var(--foreground);
}

.toggle-group {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.toggle-label {
  font-size: 0.9rem;
  color: var(--foreground);
  cursor: pointer;
}

:deep(.custom-toggle.p-toggleswitch-checked .p-toggleswitch-slider) {
  background: var(--primary);
}

.tags-wrapper { width: 100%; }

.tags-container {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
  align-items: center;
  min-height: 2.8rem;
  background: color-mix(in srgb, var(--background) 60%);
  border: 1px solid color-mix(in srgb, var(--foreground) 60%, transparent);
  border-radius: 10px;
  padding: 0.4rem 0.6rem;
  cursor: text;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.tags-container:focus-within {
  border-color: var(--primary);
}

.tag-chip {
  display: inline-flex;
  align-items: center;
  gap: 0.25rem;
  background: color-mix(in srgb, var(--foreground) 15%, transparent);
  color: var(--foreground);
  border: 1px solid color-mix(in srgb, var(--foreground) 30%, transparent);
  border-radius: 6px;
  padding: 0.15rem 0.4rem 0.15rem 0.55rem;
  font-size: 0.8rem;
  font-weight: 500;
}

.tag-remove {
  background: none;
  border: none;
  cursor: pointer;
  color: inherit;
  padding: 0;
  display: flex;
  align-items: center;
  opacity: 0.55;
  transition: opacity 0.15s;
  border-radius: 4px;
}

.tag-remove:hover { opacity: 1; }

.tag-remove .material-icons-round {
  font-size: 0.95rem;
}

.tag-input {
  border: none;
  background: transparent;
  color: var(--foreground);
  font-family: 'DM Sans', sans-serif;
  font-size: 0.88rem;
  outline: none;
  min-width: 100px;
  flex: 1;
  padding: 0.2rem 0.3rem;
}

.tag-input::placeholder {
  color: color-mix(in srgb, var(--foreground) 30%, transparent);
}

.form-divider {
  width: 100%;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  margin: 0.5rem 0;
}

.form-divider::before,
.form-divider::after {
  content: '';
  flex: 1;
  height: 1px;
  background: hsl(from var(--primary) h s calc( l * 1.1 ));
}

.divider-label {
  font-size: 0.72rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: color-mix(in srgb, var(--foreground) 70%, transparent);
  white-space: nowrap;
}

.error-message {
  font-size: 0.75rem;
  color: var(--error);
  margin: 0;
  display: flex;
  align-items: center;
  gap: 0.25rem;
}

.error-message .material-icons-round {
  font-size: 0.95rem;
}

.slide-error-enter-active { transition: all 0.2s ease; }
.slide-error-leave-active  { transition: all 0.15s ease; }
.slide-error-enter-from, .slide-error-leave-to {
  opacity: 0;
  transform: translateY(-4px);
}

:deep(.submit-btn) {
  grid-column: 1 / -1;
  width: 100%;
  background:  color-mix(in srgb, hsl(from var(--primary) h s calc( l / 2 )) 70%, transparent) ;
  border-color: var(--primary);
  color: #e2e4ff;
  text-shadow: 0 4px 12px rgba(0, 0, 0, 0.542);
  padding: 0.85rem;
  font-weight: 700;
  font-size: 0.95rem;
  font-family: inherit;
  border-radius: 12px;
  margin-top: 0.5rem;
  letter-spacing: 0.02em;
  transition: opacity 0.2s, transform 0.15s, box-shadow 0.2s;
  position: relative;
  overflow: hidden;
  transition: 200ms ease-in-out;
}

:deep(.submit-btn::after) {
  content: '';
  position: absolute;
  inset: 0;
  pointer-events: none;
}

:deep(.submit-btn:hover:not(:disabled)) {
  opacity: 0.92;
  color: #e2e4ff;
  background: var(--primary) !important;
  border-color: var(--primary) !important;
}

:deep(.submit-btn:active:not(:disabled)) {
  transform: translateY(0);
}

:deep(.submit-btn:disabled) {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>