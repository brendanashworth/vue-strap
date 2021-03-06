<template>
  <a :is="isButton?'a':'label'" @click="toggle" :class="[isButton?'btn btn-'+typeColor:'open checkbox '+typeColor,{active:checked,disabled:disabled,readonly:readonly}]">
    <input v-if="name" type="hidden" :name="name" :value="checked?trueValue:falseValue" />
    <span v-if="!isButton" class="icon dropdown-toggle" :class="[checked?'btn-'+typeColor:'',{bg:typeColor==='default'}]"></span>
    <span v-if="!isButton&&checked&&typeColor==='default'" class="icon"></span>
    <slot></slot>
  </a>
</template>

<script>
export default {
  props: {
    button: {type: Boolean, default: false},
    disabled: {type: Boolean, default: false},
    falseValue: {default: false},
    name: {type: String, default: null},
    readonly: {type: Boolean, default: false},
    trueValue: {default: true},
    type: {type: String, default: null},
    value: {default: false}
  },
  data () {
    return {
      checked: (this.value === this.trueValue)
    }
  },
  computed: {
    inGroup () { return this.$parent && this.$parent.btnGroup && !this.$parent._radioGroup },
    isButton () { return this.button || (this.$parent && this.$parent.btnGroup && this.$parent.buttons) },
    isFalse () { return this.value === this.falseValue },
    isTrue () { return this.value === this.trueValue },
    parentValue () { return this.$parent.val },
    typeColor () { return (this.type || (this.$parent && this.$parent.type)) || 'default' }
  },
  watch: {
    checked (val, old) {
      var value = val ? this.trueValue : this.falseValue
      this.$emit('checked', val)
      this.$emit('input', value);
      this.updateParent()
    },
    parentValue (val) {
      this.updateFromParent();
    },
    value (val, old) {
      var checked = val === this.trueValue
      if (this.checked !== checked) {
        this.checked = checked
      }
    }
  },
  created () {
    if (this.inGroup) {
      const parent = this.$parent
      parent._checkboxGroup = true
      if (!(parent.val instanceof Array)) { parent.val = [] }
    }
  },
  mounted () {
    this.updateFromParent();
  },
  methods: {
    // called @ mounted(), or whenever $parent.val changes
    // sync our state with the $parent.val
    updateFromParent() {
      if (this.inGroup) {
        var index = this.$parent.val.indexOf(this.trueValue)
        this.checked = ~index
      }
    },
    // called when our checked state changes
    updateParent() {
      if (this.inGroup) {
        var index = this.$parent.val.indexOf(this.trueValue)
        if (this.checked && !~index) this.$parent.val.push(this.trueValue)
        if (!this.checked && ~index) this.$parent.val.splice(index, 1)
      }
    },
    toggle () {
      if (this.disabled || this.readonly) { return }
      this.checked = !this.checked
    }
  }
}
</script>

<style scoped>
label.checkbox {
  position: relative;
  padding-left: 18px;
}
label.checkbox > input {
  box-sizing: border-box;
  position: absolute;
  z-index: -1;
  padding: 0;
  opacity: 0;
  margin: 0;
}
label.checkbox > .icon {
  position: absolute;
  top: .2rem;
  left: 0;
  display: block;
  width: 1.4rem;
  height: 1.4rem;
  line-height:1rem;
  text-align: center;
  user-select: none;
  border-radius: .35rem;
  background-repeat: no-repeat;
  background-position: center center;
  background-size: 50% 50%;
}
label.checkbox:not(.active) > .icon {
  background-color: #ddd;
  border: 1px solid #bbb;
}
label.checkbox > input:focus ~ .icon {
  outline: 0;
  border: 1px solid #66afe9;
  box-shadow: inset 0 1px 1px rgba(0,0,0,.075),0 0 8px rgba(102,175,233,.6);
}
label.checkbox.active > .icon {
  background-size: 1rem 1rem;
  background-image: url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz4NCjxzdmcgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB3aWR0aD0iNyIgaGVpZ2h0PSI3Ij48cGF0aCBmaWxsPSIjZmZmIiBkPSJtNS43MywwLjUybC0zLjEyNDIyLDMuMzQxNjFsLTEuMzM4OTUsLTEuNDMyMTJsLTEuMjQ5NjksMS4zMzY2NWwyLjU4ODYzLDIuNzY4NzZsNC4zNzM5LC00LjY3ODI2bC0xLjI0OTY5LC0xLjMzNjY1bDAsMGwwLjAwMDAyLDAuMDAwMDF6Ii8+PC9zdmc+);
}
label.checkbox.active .btn-default { filter: brightness(75%); }

label.checkbox.disabled,
label.checkbox.readonly,
.btn.readonly {
  filter: alpha(opacity=65);
  box-shadow: none;
  opacity: .65;
}
label.btn > input[type=checkbox] {
  position: absolute;
  clip: rect(0,0,0,0);
  pointer-events: none;
}
</style>
