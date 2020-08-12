<script>
class Handler {
    constructor(ctx) {
        this.ctx = ctx;
    }

    setElements(elements, currentNumber) {
        this.elements = elements;
        this.max = elements.length - 1;

        this.updateState(currentNumber);
    }

    get value() {
        return this.ctx.props.value;
    }

    input(value) {
        if (this.ctx.listeners.input) {
            this.ctx.listeners.input(value);
        }
    }

    get current() {
        return this.elements[this.currentNumber]?.elm;
    }

    get next() {
        return this.elements[this.currentNumber + 1]?.elm;
    }

    get previous() {
        return this.elements[this.currentNumber - 1]?.elm;
    }

    updateState(currentNumber = 0, focus = false) {
        this.currentNumber = Math.min(Math.max(0, currentNumber), this.max);

        if (focus) {
            this.current.focus();
        }
    }

    onFocus(e) {
        e.preventDefault();
        e.stopPropagation();

        this.current.parentNode.focus();
    }

    onKeydown(e) {
        if (e.ctrlKey || e.metaKey) {
            return;
        }

        e.preventDefault();
        e.stopPropagation();

        /**
         * Backspace
         */
        if (e.keyCode === 8) {
            if (!this.current.innerText) {
                this.updateState(this.currentNumber - 1, true);
            }

            this.input(this.value.substr(0, this.currentNumber));
            this.current.innerText = '';

            return;
        }

        /**
         * Крайний элемент с уже заполненным значением.
         * Замена значения только через стирание.
         */
        if (this.current.innerText) {
            return;
        }

        /**
         * Любой другой символ, кроме числа.
         */
        if (!/\d/.test(e.key)) {
            return;
        }

        /**
         * Число.
         */
        this.input(this.value + e.key);

        this.current.innerText = e.key;
        this.updateState(this.currentNumber + 1, true);
    }

    onPaste(e) {
        e.stopPropagation();
        e.preventDefault();

        const clipboardData = e.clipboardData || window.clipboardData;
        const pastedData = clipboardData.getData('Text').trim();

        /**
         * Можно заменить на чистку от всех символов кроме цифр.
         */
        if (/[^\d]/.test(pastedData)) {
            return;
        }

        let value = this.value.toString().substr(0, this.currentNumber);

        value = (value + pastedData).substring(0, this.max + 1);

        this.input(value);
        this.updateState(value.length, true);
    }
}

export default {
    name: 'PinCode',

    functional: true,

    props: {
        value: {
            type: [String, Number],
            required: true,
        },

        length: {
            type: Number,
            default: 4,
        },
    },

    render(h, ctx) {
        const value = ctx.props.value.toString();
        const handler = new Handler(ctx);

        const children = Array.from(Array(ctx.props.length).keys()).map(number => {
            const numberValue = value.substr(number, 1);

            return h('button', {
                class: {
                    'pin-code__item': true,
                    'pin-code__item--filled': numberValue,
                },

                on: {
                    focus: handler.onFocus.bind(handler),
                },

                domProps: {
                    innerText: numberValue,
                },
            });
        });

        handler.setElements(children, value.length);

        return h('div', {
            class: 'pin-code',

            attrs: {
                autofocus: true,
                contenteditable: true,
            },

            on: {
                paste: handler.onPaste.bind(handler),
                keydown: handler.onKeydown.bind(handler),
            },
        }, children);
    },
};
</script>

<style lang="scss">
.pin-code {
    outline: none!important;
    height: 100%;
    width: 100%;
    background-color: #000;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 40px;
    caret-color: transparent;

    &__item {
        outline: none!important;
        position: relative;
        width: 34px;
        height: 52px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 36px;
        line-height: 1.3;
        font-family: 'Roboto', serif;
        color: #fff;
        border: none;
        background: transparent;

        & + & {
            margin-left: 14px;
        }

        &::before {
            content: '';
            position: absolute;
            width: 11px;
            height: 11px;
            border-radius: 50%;
            background-color: #3C3C3C;
            top: calc(50% - 5.5px);
            left: calc(50% - 5.5px);
        }

        &::after {
            content: '';
            position: absolute;
            top: 100%;
            left: 0;
            width: 100%;
            height: 4px;
            background-color: #B8B8B8;
            transition: background-color .228s;
        }

        &--filled {
            &::after {
                background-color: #8887CD;
            }

            &::before {
                opacity: 0;
            }
        }
    }
}
</style>
