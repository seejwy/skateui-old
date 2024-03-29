<template lang='pug'>
sui-fieldset.sui-textarea(
    type="textarea"
    :class="{'sui-fieldset-disabled': $attrs['disabled'], 'required': $attrs['required']}"
    :suffix='suffix'
    :prefix='prefix'
    :label="label"
    :error="isError"
    :message="errorMessage || message")
    template(#slot-left)
        slot(name="slot-left")
    template(#slot-right)
        slot(name="slot-right")
    .wrapper
        textarea(
            ref="textarea"
            @input="updateValue()"
            @invalid.prevent="invalidInput"
            rows="1"
            @keyup="(e)=>{keyOutput(e.code)}"
            :value="value"
            @focus="focus"
            v-bind="$attrs")
</template>
<script>
export default {
    name: 'sui-textarea',
    emits: ['update:modelValue', 'input', 'focus', 'lengthError', 'error'],
    props: {
        error: Boolean,
        prefix: String,
        suffix: String,
        lengthError: String,
        label: String,
        message: String,
        readonly: Boolean,
        maxlength: {
            type: Number | String
        },
        value: {
            type: [String, Number],
            default: ''
        },
        modelValue: {
            type: [String, Number],
            default: ''
        },
        keyOutput: {
            type: Function,
            default: () => {
            }
        },
        minLengthErrorMessage: String
    },
    data() {
        return {
            errorMessage: '',
            isTouched: false,
            isError: false
        };
    },
    mounted() {
        // setup textbox replica
        let el = this.$refs.textarea;
        // let lineHeight = parseInt(document.defaultView.getComputedStyle(el, null).getPropertyValue('line-height'));
        let maxHeight = el.parentNode.parentNode.parentNode.style.maxHeight;
        if(maxHeight && el.scrollHeight >= parseInt(maxHeight)) {
            el.style.height = maxHeight;
        } else {
            el.style.height = el.scrollHeight + 'px';
        }
        // let parent = el.parentElement;
        // let replica = document.createElement('div');
        //
        // let marginSpace = ((el.parentNode.parentNode.offsetHeight - lineHeight) / 2) - 4 + 'px';
        // el.parentNode.style.marginTop = marginSpace;
        // el.parentNode.style.marginBottom = marginSpace;
        //
        // replica.classList.add('sui-textarea-replica');
        // replica.dataset.replica = el.value;
        // parent.insertBefore(replica, el);
        // replica.append(el);


        // let field = el.closest('fieldset.sui-fieldset');
        // this.inputId = field ? field.id + '_interface' : window.sui_generateId('option');
        // el.id = this.inputId;
        //
        // el.addEventListener('input', (e) => {
        //     let target = e.target;
        //     target.parentNode.dataset.replica = target.value;
        // });
        // el.addEventListener('focus', (e) => {
        //     let target = e.target;
        //     let par = target.parentNode.parentNode.parentNode;
        //     if (par.classList.contains('sui-textarea') && !par.classList.contains('focus')) {
        //         par.classList.add('focus');
        //     }
        // });
        // el.addEventListener('blur', (e) => {
        //     let target = e.target;
        //     let par = target.parentNode.parentNode.parentNode;
        //     if (par.classList.contains('sui-textarea') && par.classList.contains('focus')) {
        //         par.classList.remove('focus');
        //     }
        // });
        //
        this.$nextTick(() => {
            if(this.$refs.textarea) {
                this.$refs.textarea.addEventListener('input', (event) => {
                    if(this.$refs.textarea.checkValidity()) {
                        this.isError = false;
                        this.errorMessage = '';
                    }


                });
                this.$refs.textarea.addEventListener('invalid', (event) => {
                    event.preventDefault();
                    this.isError = true;
                    let state = this.$refs.textarea.validity;
                    for(let key in state){
                        if(state[key]) {
                            switch(key) {
                                case 'valueMissing':
                                    this.errorMessage = this.requiredErrorMessage;
                                    break;
                                case 'tooShort':
                                    this.errorMessage = this.minLengthErrorMessage;
                                    break;
                            }
                            break;
                        }
                    }
                });
            }
        });
    },
    computed: {
        lengthFail() {
            let value = this.value || this.modelValue;

            if (this.isTouched) {
                let max = parseInt(this.maxlength || 0);
                if (max && value.length > max) return true;
            }

            return false;
        },
        helperMessage() {
            let helper = this.message || null;
            return helper;
        },
    },
    methods: {
        focus(e) {
            this.$emit('focus', e);
        },
        updateValue(v) {
            let el = this.$refs.textarea;
            let maxHeight = el.parentNode.parentNode.parentNode.parentNode.style.maxHeight;
            let trueMaxHeight = parseInt(maxHeight) - parseInt(window.getComputedStyle(el.parentNode.parentNode, null).getPropertyValue('padding-top')) - parseInt(window.getComputedStyle(el.parentNode.parentNode, null).getPropertyValue('padding-bottom'));
            el.style.height = 'auto';
            if(maxHeight && el.scrollHeight > trueMaxHeight) {
                el.style.height = trueMaxHeight + 'px';
                el.style.overflowY = 'scroll';
            } else {
                el.style.height = el.scrollHeight + 'px';
                el.style.overflowY = '';
            }
            this.$emit('input', v ? v : this.$refs.textarea.value);
            this.$emit('update:modelValue', v ? v : this.$refs.textarea.value);
        },
        invalidInput() {
            this.isTouched = true;
            if (this.lengthError) {
                this.$emit('lengthError');
            } else {
                this.$emit('error');
            }
        }
    }
};
</script>