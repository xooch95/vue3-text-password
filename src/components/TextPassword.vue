<template>
  <el-input ref="passwordRef" v-bind="$attrs" v-model="modelValue">
    <template v-if="$slots.prefix" #prefix>
      <slot name="prefix"></slot>
    </template>

    <template v-if="$slots.suffix" #suffix>
      <slot name="suffix"></slot>
    </template>

    <template v-if="$slots.prepend" #prepend>
      <slot name="prepend"></slot>
    </template>

    <template v-if="$slots.append" #append>
      <slot name="append"></slot>
    </template>
  </el-input>
</template>

<script setup name="TextPassword">
import { onMounted, watch, ref, toRefs } from 'vue'
import { useFormItem } from 'element-plus'

const props = defineProps({
  modelValue: '',
  autocomplete: {
    type: Boolean,
    default: false
  },
  fixValidate: {
    type: Function,
    /**
     * 解决替换input框时，校验不正常
     * @param { FormItemContext } form 
     * @param { FormItemContext } formItem 
     */
    default: (form, formItem) => { form?.validateField([formItem?.prop]) }
  }
})

const { form, formItem } = useFormItem()

// console.log('useFormItem:', form, formItem)

const $emit = defineEmits()

const { modelValue: ciphertext, autocomplete } = toRefs(props)
const passwordRef = ref(null)

const convertToCiphertext = (val) => {
  // if (val instanceof RefImpl) {
  //   val = val.value;
  // }
  return (val || '').replace(/[^\•]/g, '•');
}

const emitModelValue = (val) => {
  $emit('update:modelValue', val);
  props.fixValidate && props.fixValidate(form, formItem)
}

const _data = {}

watch(ciphertext, (val) => {
  val = val || '';
  const coverInput = _data.plainInput;
  if (coverInput.value !== val) {
    if (autocomplete.value) {
      coverInput.value = val;
      return
    }
    coverInput.value = convertToCiphertext(val);
  }
})

onMounted(() => {
  const $display = _data.plainInput = passwordRef.value.input;
  if (autocomplete.value) {
    $display.type = 'password';
    $display.value = ciphertext;
    $display.addEventListener('input', (event) => {
      emitModelValue($display.value)
    })
    $display.addEventListener('keypress', (event) => {
      if (event.code === 'Enter' || event.code === 'NumpadEnter') {
        $emit('enter', event);
      }
    })
    return;
  }

  $display.value = convertToCiphertext(ciphertext.value);
  $display.addEventListener('focus', (event) => {
    const coverInput = $display.cloneNode();
    // coverInput.id = "";
    coverInput.type = "password";
    coverInput.value = ciphertext.value;
    $display.parentNode.insertBefore(coverInput, $display);
    $display.style.display = "none"
    coverInput.focus();

    const render = (val, cursor) => {
      coverInput.value = val;
      coverInput.setSelectionRange(cursor, cursor);
      emitModelValue(val)
    }
    coverInput.addEventListener('blur', (event) => {
      $display.value = convertToCiphertext(ciphertext.value);
      coverInput.parentNode.removeChild(coverInput);
      $display.style.display = "block"
    });
    coverInput.addEventListener('keydown', (event) => {
      // console.log('keydown', event);
      const { key } = event;
      const start = coverInput.selectionStart;
      let end = coverInput.selectionEnd;
      let $val = coverInput.value;
      coverInput.type = "password";
      coverInput.value = ciphertext.value;

      const isBackspace = key === 'Backspace';

      if (isBackspace || key === 'Delete') {
        event.preventDefault();
        if (end - start === $val.length) {
          render('', 0);
          return;
        }
        if (isBackspace) {
          if (start === 0) {
            return;
          }
          const cursor = Math.max(0, start === end ? start - 1 : start);
          render($val.substring(0, cursor) + $val.substring(end), cursor);
        } else {
          if (start === $val.length) {
            return;
          }
          end = Math.min($val.length, start === end ? end + 1 : end);
          render($val.substring(0, start) + $val.substring(end), start);
        }
      } else if (event.ctrlKey && /C|V|Z|Y|X/.test(event.key)) {
        event.preventDefault();
      }
    })

    coverInput.addEventListener('keypress', (event) => {
      // console.log('keypress', event);
      event.preventDefault();
      if (event.code === 'Enter' || event.code === 'NumpadEnter') {
        coverInput.blur();
        $emit('enter', event);
        return;
      }
      const start = coverInput.selectionStart;
      const end = coverInput.selectionEnd;
      let $val = coverInput.value;
      render($val.substring(0, start) + event.key + $val.substring(end), start + 1);
    });
  })
})

</script>
