<!-- Example of my Vue3/Nuxt3/CompositionApi abilities -->
<template>
  <div
    :class="{
      [$style.host]: true,
      [$style.hostHasErrors]: errors
    }"
  >
    <input
      v-for="(_, pos) in numberOfFields"
      v-model="values[pos]"
      :ref="(el) => {refs['input' + pos] = el}"
      :key="id + pos"
      :class="$style.input"
      :maxlength="fieldMaxLength"
      autocomplete="off"
      @paste="onPaste(pos, $event)"
      @keydown="onKeydown(pos, $event)"
    />
  </div>
</template>
<style module lang="scss">
.host {
  display: inline-block;
}

.input {
  line-height: 3em;
  text-align: center;

  width: 3em;
  height: 3em;
  box-sizing: border-box;

  border: gray solid 1px;
}

.hostHasErrors .input {
  border: red solid 1px;
}

.hostIsCorrect .input {
  border: green solid 1px;
}
</style>
<script setup lang="ts">
const EVENT_UPDATE_LOCALE = 'update:modelValue'

interface InputOTPProps {
  id?: number | string
  groupBy?: number
  fieldMaxLength?: number
  numberOfFields?: number
  modelValue?: string
}

interface RefInputs<Value = HTMLInputElement | null> {
  [id: string]: Value
}

type TimerId = ReturnType<typeof setTimeout | typeof setInterval | null>

const props = withDefaults(defineProps<InputOTPProps>(), {
  id: 'input-otp',
  groupBy: 4,
  fieldMaxLength: 1,
  numberOfFields: 4,
  modelValue: ''
})
const emit = defineEmits([EVENT_UPDATE_LOCALE])

const refs = <RefInputs>{}
const errors = ref<boolean | string>(false)
const values = ref<string[]>([])
let timer = <TimerId>null

for (let it0 = 0, ln0 = props.numberOfFields; it0 < ln0; it0++) {
  values.value.push('')

  refs[`input${it0}`] = null

  watch(() => values.value[it0], onInputChange)
}

function setErrors() {
  errors.value = true
}

function resetErrors() {
  errors.value = false
}

function emitValue(val: string) {
  if (timer) {
    clearTimeout(timer)
  }

  timer = setTimeout(() => {
    emit(EVENT_UPDATE_LOCALE, val)
  }, 50)
}

function checkValue(raw: string): boolean {
  resetErrors()

  const pattern = new RegExp(`^[0-9a-zA-Z]{${props.numberOfFields}}$`)

  return pattern.test(raw)
}

function compileValue(): string {
  let value = ''

  for (let it0 = 0, ln0 = props.numberOfFields; it0 < ln0; it0++) {
    value += values.value[it0]
  }

  return value
}

function onPaste(pos: number, event: ClipboardEvent) {
  // No need to go further
  if (!event.clipboardData) {
    return
  }

  // To prevent default letter input in focused field
  event.preventDefault()

  let val = event.clipboardData.getData('text/plain')

  // No need to go further
  if (val.length > props.numberOfFields) {
    errors.value = 'tooLong'
    return
  }

  // Remove spaces and line breaks
  val = val.replace(/[\s\t\n\r]+/g, '')

  if (val.length === 1) {
    if (!/[0-9a-zA-Z]/.test(val)) {
      errors.value = 'wrongSymbol'
    }

    values.value[pos] = val

    switchToNextInput(pos)
  } else {
    for (let it0 = 0, ln0 = props.numberOfFields; it0 < ln0; it0++) {
      if (val[it0] && !/[0-9a-zA-Z]/.test(val[it0])) {
        errors.value = 'wrongSymbol'
      }

      values.value[it0] = val[it0] || ''
    }

    switchToNextInput(val.length - 1)
  }
}

function onKeydown(pos: number, event: KeyboardEvent) {
  const code = event.code || event.keyCode
  const input = refs[`input${pos}`] as HTMLInputElement
  // const { selectionStart, selectionEnd } = input

  switch (code) {

    case 8:
    case 'Backspace':
      if (input.value.length === 0) {
        event.preventDefault()

        switchToPrevInput(pos)
      }
      break
    case 37:
    case 'ArrowLeft':
      switchToPrevInput(pos)
      break

    case 39:
    case 'ArrowRight':
      switchToNextInput(pos)
      break

  }
}

function onInputChange() {
  const val = compileValue()

  emitValue(val)

  switchToNextInput(val.length - 1)

  if (!checkValue(val) && val.length === props.numberOfFields) {
    setErrors()
  }
}

function switchToPrevInput(pos: number) {
  const prev = refs[`input${pos - 1}`]
  const input = refs[`input${pos}`] as HTMLInputElement
  const { selectionEnd } = input

  if (!prev || selectionEnd !== 0) {
    return
  }

  prev.focus()

  setTimeout(() => {
    prev.setSelectionRange(prev.value.length, prev.value.length);
  }, 0)
}

function switchToNextInput(pos: number) {
  const next = refs[`input${pos + 1}`]
  const input = refs[`input${pos}`] as HTMLInputElement
  const { selectionEnd } = input

  if (!next || selectionEnd !== input.value.length) {
    return
  }

  next.focus()

  setTimeout(() => {
    next.setSelectionRange(0, 0);
  }, 0)
}

</script>
