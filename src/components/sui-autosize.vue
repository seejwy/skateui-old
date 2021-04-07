<template lang="pug">
.sui-autosize(:id="elementId")
    textarea(:placeholder='placeholder' rows="1" v-model="inputValue" :maxlength="maxlength")
</template>

<script>
export default {
    name: "sui-autosize",
    props: {
        placeholder: String,
        min: Number,
        max: Number,
        value: String,
        output: Function,
        allowEnter: Boolean,
        maxlength: Number
    },
    data() {
        return {
            autosize: null
        };
    },
    created() {
        class SuiAutosize {
            constructor(el) {
                if (!el)
                    throw 'no argument';
                this.eventId = null;
                this.max = 72;
                this.min = 14;
                this.value = '';
                this.allowEnter = false;
                if (typeof el === 'string')
                    this.element = document.getElementById(el[0] === '#' ? el.substring(1) : el);
                else if (typeof el === 'object' && Object.keys(el).length) {
                    let {element, elementId, max, min, value, allowEnter} = el;

                    this.value = value || "";

                    if (allowEnter)
                        this.allowEnter = allowEnter;

                    if (typeof elementId === 'string')
                        this.element = document.getElementById(elementId[0] === '#' ? elementId.substring(1) : elementId);

                    else if (element && element instanceof Node) this.element = element;

                    else
                        throw 'no element';

                    this.max = max || 72;
                    this.min = min || 14;
                }

                if (!this.element)
                    throw 'no element';


                this.textarea = this.element.childNodes[0];
                this.elementStyle = window.getComputedStyle(this.textarea);
                this.init();
            }

            updateValue(v) {
                let target = v || this.textarea;
                target.parentNode.dataset.replica = target.value;
                this.value = target.value;
                this.adjustSize();
            }

            init() {
                let el = this.textarea;
                let parent = this.element;
                el.value = this.value;

                el.setAttribute('rows', '1');

                if (el.readOnly && !parent.classList.contains('readonly'))
                    parent.classList.add('readonly');

                let replica = document.createElement('div');
                replica.classList.add('textarea');
                parent.insertBefore(replica, el);
                replica.append(el);

                replica.dataset.replica = el.value;

                el.addEventListener('keydown', (e) => {
                    if (!this.allowEnter && e.key === 'Enter')
                        e.preventDefault();
                });

                el.addEventListener('input', (e) => {
                    this.updateValue();
                });

                el.addEventListener('focus', (e) => {
                    let target = e.target;
                    let par = target.parentNode.parentNode;
                    if (!target.readOnly && par.classList.contains('sui-autosize') && !par.classList.contains('focus'))
                        par.classList.add('focus');
                });

                el.addEventListener('blur', (e) => {
                    let target = e.target;
                    let par = target.parentNode.parentNode;
                    if (!target.readOnly && par.classList.contains('sui-autosize') && par.classList.contains('focus'))
                        par.classList.remove('focus');
                });

                this.adjustSize();
                if (window.sui_app)
                    this.eventId = window.sui_app.registerEvent.resize(this.adjustSize.bind(this));
            }

            adjustSize() {

                if (!this.fontsize) {
                    let width = parseFloat(this.elementStyle.width);
                    let charLength = this.textarea.value.length;
                    let div = width / charLength;

                    div = div * 1.33;

                    if (this.max < div)
                        this.fontsize = this.max;

                    else if (this.min > div)
                        this.fontsize = this.min;

                    else
                        this.fontsize = div;
                }

                let runDown = () => {
                    this.element.style.setProperty('--auto-size', `${this.fontsize}px`);
                    let doIt = async () => {
                        let height = parseFloat(this.elementStyle.height);
                        let howmanylines = height / (this.fontsize * 1.5);
                        if (howmanylines > 2 && this.fontsize > this.min) {
                            let minus = this.fontsize - 1;
                            this.fontsize = minus > this.min ? minus : this.min;
                            await runDown();
                        }
                    };

                    return new Promise(res => {
                        doIt();
                        res(true);
                    });
                };

                let runUp = () => {
                    this.element.style.setProperty('--auto-size', `${this.fontsize}px`);

                    let doIt = async () => {
                        let height = parseFloat(this.elementStyle.height);
                        let howmanylines = height / (this.fontsize * 1.5);
                        if (howmanylines < 2 && this.fontsize < this.max) {
                            let plus = this.fontsize + 1;
                            this.fontsize = plus < this.max ? plus : this.max;
                            await runUp();
                        } else {
                            await runDown();
                        }
                    };

                    return new Promise(res => {
                        doIt();
                        res(true);
                    });
                };

                runUp();
            }

            destroy() {
                if (window.sui_app)
                    window.sui_app.removeEvent.resize(this.eventId);
            }
        }

        if (!window.sui_autosize)
            window.sui_autosize = SuiAutosize;
    },
    mounted() {
        this.autosize = new window.sui_autosize({
            elementId: this.elementId,
            min: this.min,
            max: this.max,
            value: this.value,
            allowEnter: this.allowEnter
        });
    },
    beforeDestroy() {
        this.autosize.destroy();
    },
    computed: {
        elementId() {
            function generateId(option) {
                let limit = 12;
                let prefix = '';

                if (typeof option === 'number') limit = option;
                else if (typeof option === 'string') prefix = `${option}_`;

                const possible = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz';

                let text = '';
                for (let i = 0; i < limit - 6; i++) text += possible.charAt(Math.floor(Math.random() * (possible.length - 1)));

                const numb = new Date()
                    .getTime()
                    .toString()
                    .substring(7, 13); // SECOND, MILLISECOND

                const shuffleArray = (array) => {
                    let currentIndex = array.length;
                    let temporaryValue, randomIndex;
                    while (0 !== currentIndex) {
                        randomIndex = Math.floor(Math.random() * currentIndex);
                        currentIndex -= 1;
                        temporaryValue = array[currentIndex];
                        array[currentIndex] = array[randomIndex];
                        array[randomIndex] = temporaryValue;
                    }
                    return array;
                };

                const letter_array = shuffleArray((text + numb).split(''));

                let output = '';
                for (let i = 0; i < limit; i++) output += letter_array[i];

                return prefix + output;
            }

            return generateId(this.$options.name);
        },
        inputValue: {
            get: function () {
                if (this.autosize)
                    this.autosize.updateValue(this.value);

                return this.value;
            },
            set: function (v) {
                if (typeof this.output === 'function')
                    this.output(v);
            }
        }
    }
};
</script>

<style lang="less">
.sui-autosize {
    position: relative;
    border: 2px dashed transparent;
    font-size: var(--auto-size);
    display: inline-block;

    &:not(.readonly) {
        border-color: var(--content-text_shadow);

        &.focus, &:hover {
            border-color: var(--content-placeholder);
        }
    }

    & > .textarea {
        position: relative;
        vertical-align: middle;
        display: inline-grid;
        min-height: 2rem;
        width: 100%;

        &::after {
            content: attr(data-replica) " ";
            white-space: pre-wrap;
            color: var(--content-text);
        }

        & > textarea {
            position: absolute;
            top: 0;
            color: transparent;
            caret-color: var(--content-text);
            resize: none;
            overflow: hidden;
            min-height: 1em;
            z-index: 1;
            border: none;
            height: 100%;
            width: 100%;

            &::placeholder {
                color: var(--content-placeholder);
            }

            &:read-only {
                text-overflow: ellipsis;
            }
        }

        & > textarea,
        &::after {
            /* Identical styling required!! */
            box-sizing: border-box;
            vertical-align: middle;
            background-color: transparent;
            line-height: 1.5em;
            font-size: 1em;
            padding: .5rem 0.75rem;
            outline: none;
            border: none;
            max-width: 100%;
            /* Place on top of each other */
            //grid-area: 1 e("/") 1 e("/") 2 e("/") 2;
        }
    }
}
</style>